# specification/v1_0/eval/src/generation_flow.ts

## 개요

UI 컴포넌트 JSON을 생성하는 핵심 Genkit flow를 정의한다. 사용자 프롬프트, 모델 설정, 프로토콜 스키마를 입력으로 받아 지정된 AI 모델에 상세한 시스템 지침과 함께 프롬프트를 전달하고, 생성된 텍스트와 레이턴시를 반환한다. 토큰 추정 기반 레이트 리미팅과 실제 토큰 사용량 사후 조정을 모두 수행한다.

## 의존성

### 외부 패키지
- `genkit` — `z` (Zod 스키마)

### 저장소 내부 모듈
- [`./ai`](./ai.ts.md)
- [`./models`](./models.ts.md) — `ModelConfiguration` 타입
- [`./rateLimiter`](./rateLimiter.ts.md)
- [`./logger`](./logger.ts.md)
- [`./types`](./types.ts.md) — `ProtocolSchemas` 타입

## Exports

| 이름 | 종류 | 설명 |
|------|------|------|
| `componentGeneratorFlow` | Genkit Flow | 프롬프트와 스키마를 받아 UI JSON 텍스트를 생성하는 flow |

## 상세 명세

### `componentGeneratorFlow` (Genkit Flow, export)

`ai.defineFlow(config, handler)`로 정의된 flow.

**입력 스키마 (`z.object`):**
- `prompt`: `string` — 사용자의 UI 생성 요청 텍스트
- `modelConfig`: `z.custom<ModelConfiguration>()` — 모델 설정 객체 (모델 참조, API 설정 포함)
- `schemas`: `z.custom<ProtocolSchemas>()` — UI 프로토콜 JSON 스키마 맵
- `catalogRules`: `z.string().optional()` — 카탈로그별 추가 지침 문자열

**출력 스키마:** `z.any()` — `{ text: string, latency: number }` 객체

**핸들러 동작 단계:**

1. `schemas`의 모든 값을 `JSON.stringify(s, null, 2)`로 직렬화하여 `schemaDefs` 문자열을 만든다.

2. 16개의 표준 지침과 스키마 정의, 사용자 요청을 포함하는 `fullPrompt`를 조립한다. 핵심 지침 내용:
   - `createSurface` 메시지를 `surfaceId: 'main'`, `catalogId: 'https://a2ui.org/specification/v1_0/catalogs/basic/catalog.json'`으로 생성할 것
   - `updateComponents` 메시지를 `surfaceId: 'main'`으로 생성할 것
   - 자식 컴포넌트는 반드시 ID 문자열로 참조 (인라인 중첩 금지)
   - 필요 시 `updateDataModel` 메시지도 생성 가능
   - 반드시 `id: 'root'`인 루트 컴포넌트가 하나 존재해야 함
   - 루트는 Column 또는 Row 레이아웃 컨테이너여야 함
   - 고아 컴포넌트 금지
   - 리스트의 리스트(`[[...]]`) 출력 금지
   - JSON 스키마를 엄격히 준수, 미정의 프로퍼티 추가 금지
   - 스키마의 `description` 필드를 반드시 참조할 것
   - `version: 'v1.0'` 프로퍼티를 모든 메시지 최상위에 포함할 것
   - `catalogRules`가 있으면 마지막에 카탈로그별 지침으로 추가

3. `fullPrompt.length / 2.5`를 올림하여 `estimatedInputTokens`를 계산한다.

4. `rateLimiter.acquirePermit(modelConfig, estimatedInputTokens)`를 await한다.

5. `Date.now()`로 시작 시각 기록 후 `ai.generate({ prompt: fullPrompt, model: modelConfig.model, config: modelConfig.config })`를 호출한다. 오류 발생 시 `rateLimiter.reportError`를 호출하고 예외를 re-throw한다.

6. 응답 구조를 두 가지 방식으로 처리한다:
   - `response.candidates?.[0]`이 있으면 `candidate`로 사용
   - 없지만 `response.message`가 있으면 `{ index: 0, content: message.content, finishReason: 'STOP', message }` 구조로 `candidate`를 구성 (Genkit 0.9+ 또는 특정 어댑터 호환성 폴백)
   - `candidate`가 없으면 오류를 throw

7. `candidate.finishReason`이 `'STOP'`도 `undefined`도 아니면 경고를 기록한다.

8. 실제 토큰 사용량(`response.usage?.inputTokens`, `response.usage?.outputTokens`)을 읽어 추정치와의 차이를 계산한다. `additionalInputTokens = Math.max(0, inputTokens - estimatedInputTokens)`, `tokensToAdd = additionalInputTokens + outputTokens`. `tokensToAdd > 0`이면 `rateLimiter.recordUsage(modelConfig, tokensToAdd, false)`를 호출하여 실제 사용량을 사후 반영한다.

9. `{ text: response.text, latency: Date.now() - startTime }`를 반환한다.

## 동작 흐름

`Generator.runJob`에서 각 (모델, 프롬프트, 실행 번호) 조합에 대해 호출된다. flow는 시스템 지침이 포함된 전체 프롬프트를 조립하고, 레이트 리미터로 속도를 제어하며 AI 모델을 호출한다. 반환된 원시 텍스트는 `Generator`에서 `extractJsonFromMarkdown`으로 파싱된다.
