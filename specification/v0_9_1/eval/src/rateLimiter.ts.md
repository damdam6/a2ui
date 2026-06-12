# specification/v0_9_1/eval/src/rateLimiter.ts

## 개요

이 파일은 LLM API 호출 시 발생할 수 있는 RPM(분당 요청 수)·TPM(분당 토큰 수) 초과를 방지하기 위한 `RateLimiter` 클래스와 그 싱글턴 인스턴스를 정의한다. 슬라이딩 윈도우 방식으로 최근 65초간의 사용 기록을 유지하며, 429 오류 발생 시 모델별로 일정 시간 동안 호출을 전면 차단(circuit-breaker 패턴)하는 기능도 포함한다. 여러 병렬 요청이 동시에 허가를 요청하더라도 permit 획득 시 즉시 기록을 삽입하여 경쟁 조건을 최소화한다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈
- [`./logger`](./logger.ts.md) — `logger` (verbose/debug 레벨 로깅)
- [`./models`](./models.ts.md) — `ModelConfiguration` (타입 참조)

## Exports

| 이름 | 종류 |
|---|---|
| `RateLimiter` | 클래스 |
| `rateLimiter` | `RateLimiter` 상수 (싱글턴 인스턴스) |

## 상세 명세

### 인터페이스 (모듈 private)

**`UsageRecord`**
| 필드 | 타입 | 설명 |
|---|---|---|
| `timestamp` | `number` | 기록 생성 시각 (`Date.now()` 밀리초) |
| `tokensUsed` | `number` | 이 기록이 소비한 토큰 수 |
| `isRequest` | `boolean` | 요청 기록 여부 (RPM 카운팅에 사용) |

**`ModelRateLimitState`**
| 필드 | 타입 | 설명 |
|---|---|---|
| `usageRecords` | `UsageRecord[]` | 모델별 슬라이딩 윈도우 사용 기록 배열 |

---

### `RateLimiter` 클래스 (export)

#### 필드

| 필드 | 타입 | 접근 | 설명 |
|---|---|---|---|
| `modelStates` | `Map<string, ModelRateLimitState>` | private | 모델 이름 → 사용 기록 상태 |
| `_waitingCount` | `number` | private | 현재 `acquirePermit`를 기다리고 있는 호출 수 |
| `modelPauses` | `Map<string, number>` | private | 모델 이름 → pause 종료 시각 (Unix ms) |

#### 게터

**`waitingCount: number`**
`_waitingCount`를 읽기 전용으로 노출한다. 진행 표시줄 등 외부에서 대기 중인 요청 수를 표시하는 데 사용된다.

#### private 메서드

**`getModelState(modelName: string): ModelRateLimitState`**
`modelStates` 맵에서 `modelName`에 해당하는 상태 객체를 반환한다. 해당 키가 없으면 `{usageRecords: []}` 를 새로 삽입한 뒤 반환한다.

**`cleanUpRecords(state: ModelRateLimitState): void`**
`Date.now() - 65 * 1000` (65초 전) 이전의 기록을 `state.usageRecords`에서 제거한다. 65초를 사용하는 이유는 서버의 버킷 경계 정렬 및 클럭 드리프트에 대한 안전 마진 확보다.

#### public 메서드

**`reportError(modelConfig: ModelConfiguration, error: any): void`**

429(RESOURCE_EXHAUSTED) 오류를 감지하여 해당 모델을 일정 시간 동안 차단한다.

동작 단계:
1. `error.status === 'RESOURCE_EXHAUSTED'` 이거나 `error.code === 429` 이거나 `error.message`에 `'429'`가 포함되면 레이트 리밋 오류로 판단한다.
2. `error.originalMessage || error.message`에서 `retry in ([0-9.]+)\s*s` 정규식으로 재시도 대기 시간(초)을 파싱한다.
3. 파싱 실패 시 기본값 `60`초를 사용한다.
4. `pauseDuration = Math.ceil(retrySeconds * 1000) + 1000` (1초 버퍼 추가)로 차단 시간을 계산한다.
5. `modelPauses.set(modelConfig.name, Date.now() + pauseDuration)`으로 차단 종료 시각을 기록한다.
6. `logger.verbose`로 차단 내용을 로깅한다.

**`acquirePermit(modelConfig: ModelConfiguration, tokensCost: number = 0): Promise<void>`**

비동기 메서드. 호출 즉시 `_waitingCount`를 증가시키고, `finally` 블록에서 감소시킨다.

동작 단계 (무한 루프 내에서 조건 충족 시 break):
1. `modelConfig.requestsPerMinute`와 `modelConfig.tokensPerMinute`가 모두 falsy면 즉시 반환한다(무제한).
2. `modelPauses`에서 현재 모델의 차단 종료 시각을 확인한다. 아직 차단 중이면 남은 시간만큼 `setTimeout`으로 대기한 후 루프를 처음부터 다시 시작한다.
3. `cleanUpRecords`를 호출하여 65초 초과 기록을 제거한다.
4. `usageRecords`를 순회하여 `currentTokens`(전체 토큰 합)와 `currentRequests`(`isRequest === true` 개수)를 집계한다.
5. TPM에 10% 안전 버퍼를 적용: `effectiveTokensPerMinute = Math.floor(tokensPerMinute * 0.9)`.
6. RPM 검사: `currentRequests + 1 > requestsPerMinute`이면, `usageRecords`에서 가장 오래된 `isRequest === true` 기록을 찾아 `rpmWait = oldestRequest.timestamp + 60_000 - currentNow`를 계산한다.
7. TPM 검사: `currentTokens + tokensCost > effectiveTokensPerMinute`이면, 기록을 오래된 순으로 누적하면서 제거해야 할 토큰(`tokensToShed = currentTokens + tokensCost - effectiveTokensPerMinute`)을 충족하는 최초 기록의 만료 시각으로 `tpmWait`를 계산한다.
8. `requiredWait = Math.max(rpmWait, tpmWait)`. 0 이하면 즉시 `usageRecords`에 permit 예약 기록(`{timestamp, tokensUsed: tokensCost, isRequest: true}`)을 삽입하고 `break`한다. 양수면 `logger.verbose`로 로깅 후 `setTimeout`으로 대기하고 루프를 반복한다.

**`recordUsage(modelConfig: ModelConfiguration, tokensUsed: number, isRequest: boolean = true): void`**

실제 API 응답 후 토큰 사용량을 기록한다. `tokensUsed > 0` 이거나 `isRequest === true`인 경우에만 `usageRecords`에 현재 타임스탬프와 함께 추가한다.

> 주의: `acquirePermit`에서 이미 permit 예약 기록을 삽입하므로, `recordUsage`는 응답 후 실제 토큰 수를 보완적으로 기록하는 용도로 사용된다.

---

### `rateLimiter` (export)

`new RateLimiter()`로 생성된 싱글턴 인스턴스. 평가 파이프라인 전체에서 공유된다.

## 동작 흐름

평가 파이프라인의 생성 단계에서 각 (모델, 프롬프트) 쌍에 대해 병렬로 `acquirePermit(modelConfig, estimatedTokens)`를 호출한다. permit를 획득한 호출만 실제 LLM API 요청을 진행한다. 응답 수신 후 `recordUsage`로 실제 소비 토큰을 기록한다. 429 오류 발생 시 `reportError`를 호출하면 해당 모델 전체가 일정 시간 동안 차단되고, 이후 `acquirePermit` 호출들은 차단이 풀릴 때까지 자동으로 대기한다.
