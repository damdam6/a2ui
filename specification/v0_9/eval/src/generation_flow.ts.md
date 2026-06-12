# specification/v0_9/eval/src/generation_flow.ts

## 개요

UI 컴포넌트 JSON을 생성하는 핵심 Genkit 플로우(`componentGeneratorFlow`)를 정의한다. 사용자 요청 프롬프트, 모델 설정, JSON 스키마, 카탈로그 규칙을 입력으로 받아 UI 프로토콜에 맞는 JSON 메시지 스트림을 텍스트로 생성한다. 레이트 리미터를 통한 토큰 사용 제어와 실제 사용량 기반 사후 보정을 수행한다.

## 의존성

### 외부 패키지

- `genkit` — `z` (Zod 스키마)

### 저장소 내부 모듈

- [`./ai`](./ai.ts.md) — `ai` 인스턴스
- [`./models`](./models.ts.md) — `ModelConfiguration` 타입
- [`./rateLimiter`](./rateLimiter.ts.md) — `rateLimiter` 인스턴스
- [`./logger`](./logger.ts.md) — `logger` 인스턴스

## Exports

- `componentGeneratorFlow` — Genkit 플로우 객체 (상수)

## 상세 명세

### `componentGeneratorFlow` (Genkit 플로우)

`ai.defineFlow(config, handler)`로 정의된 플로우이다.

**플로우 이름:** `'componentGeneratorFlow'`

**inputSchema:**
- `prompt`: `z.string()` — 사용자 UI 생성 요청 텍스트
- `modelConfig`: `z.any()` — `ModelConfiguration` 객체 (Zod 타입 미정의)
- `schemas`: `z.any()` — JSON 스키마 객체 맵
- `catalogRules`: `z.string().optional()` — 카탈로그별 추가 지시사항

**outputSchema:** `z.any()` — `{ text: string, latency: number }` 객체

**핸들러 동작:**

1. `schemas`의 모든 값을 `JSON.stringify(..., null, 2)`로 직렬화하여 줄바꿈으로 이어 붙인 `schemaDefs`를 만든다.
2. `fullPrompt` 문자열을 구성한다. 이 프롬프트는 다음을 포함한다:
   - 역할: `"You are an AI assistant."`
   - 출력 형식: Markdown 코드 블록으로 감싼 JSON 객체 시리즈
   - 표준 지시사항 15개 항목. 주요 사항:
     - `createSurface` 메시지는 `surfaceId: 'main'`, `catalogId: 'https://a2ui.org/specification/v0_9/catalogs/basic/catalog.json'`으로 생성
     - `updateComponents` 메시지에 `surfaceId: 'main'` 사용
     - 자식 컴포넌트는 인라인 중첩 금지, 반드시 문자열 ID로 참조
     - `root` ID를 가진 루트 컴포넌트 필수
     - 루트는 Column 또는 Row 레이아웃 컨테이너여야 함
     - 모든 메시지에 `"version": "v0.9"` 최상위 속성 포함 필수
     - 정의되지 않은 속성 추가 금지, `style` 속성 사용 금지
   - `catalogRules`가 있으면 `\nInstructions specific to this catalog:\n${catalogRules}` 추가
   - `schemaDefs` 및 `prompt` 포함
3. `fullPrompt.length / 2.5`를 올림하여 `estimatedInputTokens`를 계산한다.
4. `rateLimiter.acquirePermit(modelConfig, estimatedInputTokens)`로 요청 허가를 기다린다.
5. 생성 시작 시간 `startTime`을 기록한다.
6. `ai.generate({ prompt: fullPrompt, model: modelConfig.model, config: modelConfig.config })`를 호출한다. 오류 발생 시 `logger.error`로 기록하고, `rateLimiter.reportError`를 호출한 뒤 예외를 재발생시킨다.
7. `latency = Date.now() - startTime`을 계산한다.
8. 응답이 없으면 `'Failed to generate component'` 오류를 던진다.
9. 응답에서 `candidates[0]`을 추출한다. 없으면 `response.message`를 사용하여 candidate 객체를 직접 구성한다 (`finishReason: 'STOP'`로 가정).
10. candidate가 없으면 전체 응답을 로그에 기록하고 `'No candidates returned'` 오류를 던진다.
11. `candidate.finishReason`이 `'STOP'`이 아니고 `undefined`도 아니면 경고를 기록한다.
12. 실제 `inputTokens`와 `outputTokens`를 `response.usage`에서 추출한다. 추정치와의 차이(`additionalInputTokens`)와 출력 토큰을 합산하여 `tokensToAdd`를 계산한다. 양수이면 `rateLimiter.recordUsage(..., false)`로 사후 보정한다.
13. `{ text: response.text, latency }`를 반환한다.

## 동작 흐름

`Generator.runJob`에서 호출됨 → 프롬프트 조립 → 레이트 리미터 허가 → LLM 텍스트 생성 → 토큰 사용량 사후 보정 → `{ text, latency }` 반환.
