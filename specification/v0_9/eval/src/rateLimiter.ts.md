# specification/v0_9/eval/src/rateLimiter.ts

## 개요

모델별 RPM(분당 요청 수)·TPM(분당 토큰 수) 기반의 레이트 리밋을 구현하는 파일이다. `RateLimiter` 클래스는 슬라이딩 윈도우 방식으로 최근 65초 내의 사용 기록을 추적하며, 한도 초과 시 필요한 시간만큼 비동기 대기한다. 429 에러 발생 시 해당 모델을 지정 시간 동안 일시 정지하는 서킷 브레이커 기능도 포함한다. `rateLimiter` 싱글턴을 export하여 전역에서 공유된다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈
- [`./logger`](./logger.ts.md) — `logger` (verbose/debug 레벨 로깅)
- [`./models`](./models.ts.md) — `ModelConfiguration` 타입 (매개변수로 사용)

## Exports

| 이름 | 종류 | 설명 |
|------|------|------|
| `RateLimiter` | 클래스 | 레이트 리밋 관리 클래스 |
| `rateLimiter` | 상수 (RateLimiter 인스턴스) | 전역 공유 싱글턴 |

## 상세 명세

### 비공개 인터페이스

#### `UsageRecord`
슬라이딩 윈도우 내의 단일 사용 기록.
- `timestamp: number` — `Date.now()` 기준 밀리초
- `tokensUsed: number` — 해당 기록에서 소비된 토큰 수
- `isRequest: boolean` — 요청 카운트 여부 (RPM 계산에 사용)

#### `ModelRateLimitState`
모델별 상태 컨테이너.
- `usageRecords: UsageRecord[]` — 시간순으로 쌓이는 사용 기록 배열

### `RateLimiter` 클래스 (export)

#### 필드

| 필드 | 타입 | 초기값 | 설명 |
|------|------|--------|------|
| `modelStates` | `Map<string, ModelRateLimitState>` | 빈 Map | 모델 이름을 키로 하는 상태 맵 |
| `_waitingCount` | `number` | `0` | 현재 permit 대기 중인 호출 수 |
| `modelPauses` | `Map<string, number>` | 빈 Map | 모델 이름 → 재개 시각(ms) 맵 |

#### `get waitingCount(): number`
`_waitingCount`를 읽기 전용으로 노출하는 getter.

#### `private getModelState(modelName: string): ModelRateLimitState`
`modelStates`에 `modelName` 키가 없으면 `{usageRecords: []}` 를 생성하여 저장한 뒤 반환한다.

#### `private cleanUpRecords(state: ModelRateLimitState): void`
현재 시각으로부터 65,000ms(65초) 이전 기록을 `state.usageRecords`에서 필터링하여 제거한다. 65초를 사용하는 이유는 서버의 버킷 정렬 및 클럭 드리프트에 대한 안전 마진 확보다.

#### `reportError(modelConfig: ModelConfiguration, error: any): void`
429/RESOURCE_EXHAUSTED 에러를 수신했을 때 호출하는 메서드.

1. `error.status === 'RESOURCE_EXHAUSTED'`, `error.code === 429`, 또는 `error.message`에 `'429'`가 포함되면 `isResourceExhausted` 플래그를 `true`로 설정한다.
2. `isResourceExhausted`가 참이면 `error.originalMessage` 또는 `error.message`에서 `/retry in ([0-9.]+)\s*s/i` 정규식으로 재시도 대기 초를 파싱한다. 파싱 실패 시 기본값 60초를 사용한다.
3. 대기 시간을 `Math.ceil(retrySeconds * 1000) + 1000` ms로 계산한다(1초 버퍼 추가).
4. `Date.now() + pauseDuration`을 `modelPauses`에 저장하고 verbose 로그를 남긴다.

#### `async acquirePermit(modelConfig: ModelConfiguration, tokensCost: number = 0): Promise<void>`
실제 모델 호출 전에 반드시 호출해야 하는 permit 획득 메서드. `_waitingCount`를 먼저 증가시키고, finally 블록에서 반드시 감소시킨다.

1. `requestsPerMinute`와 `tokensPerMinute`가 모두 falsy이면 즉시 반환(무제한).
2. 모델 상태를 가져와 무한 루프를 시작한다.
   - **서킷 브레이커 확인**: `modelPauses`에서 해당 모델의 `pausedUntil` 값을 읽어, 현재 시각보다 크면 `pausedUntil - Date.now()` 만큼 `setTimeout`으로 대기한 뒤 루프를 `continue`한다.
   - `cleanUpRecords`로 오래된 기록을 제거한다.
   - 현재 윈도우의 총 토큰(`currentTokens`)과 요청 수(`currentRequests`)를 `usageRecords` 합산으로 계산한다.
   - **RPM 확인**: `currentRequests + 1 > requestsPerMinute`이면, 가장 오래된 요청 기록의 `timestamp + 60,000 - 현재` 를 `rpmWait`로 설정한다.
   - **TPM 확인**: `tokensPerMinute`가 있으면 10% 안전 마진을 적용한 `effectiveTokensPerMinute = Math.floor(tokensPerMinute * 0.9)`를 계산한다. `currentTokens + tokensCost > effectiveTokensPerMinute`이면 오래된 기록부터 누적하여 한도 초과분이 해소될 때까지의 기록 타임스탬프 기준 대기 시간을 `tpmWait`로 설정한다.
   - `requiredWait = Math.max(rpmWait, tpmWait)`. `requiredWait <= 0`이면 현재 시각·`tokensCost`·`isRequest: true`로 예약 기록을 `usageRecords`에 추가하고 루프를 `break`한다(레이스 컨디션 방지를 위해 대기 없이 즉시 예약).
   - `requiredWait > 0`이면 verbose 로그를 남기고 `setTimeout`으로 대기한 뒤 루프를 다시 시작한다.

#### `recordUsage(modelConfig: ModelConfiguration, tokensUsed: number, isRequest: boolean = true): void`
실제 응답에서 얻은 토큰 수를 사후 기록하는 메서드. `tokensUsed > 0` 이거나 `isRequest`가 `true`일 때만 `usageRecords`에 현재 타임스탬프와 함께 추가한다. `acquirePermit`에서 이미 예약 기록이 추가되므로 이 메서드는 보조적 실제 사용량 기록 용도이다.

### `rateLimiter: RateLimiter` (export)
`new RateLimiter()`로 생성한 모듈 수준 싱글턴. 모든 모델·모든 프롬프트 실행에 걸쳐 공유된다.

## 동작 흐름

평가 파이프라인에서 모델을 호출하기 전에 `rateLimiter.acquirePermit(modelConfig, estimatedTokens)`를 await하여 한도 범위 안에서 실행 권한을 획득한다. 호출 성공 후 `rateLimiter.recordUsage`로 실제 토큰 사용량을 보강 기록한다. 429 에러 수신 시 `rateLimiter.reportError`로 서킷 브레이커를 설정한다. 진행 표시줄 등에서 `rateLimiter.waitingCount`를 읽어 대기 중인 요청 수를 표시한다.
