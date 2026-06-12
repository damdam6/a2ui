# specification/v1_0/eval/src/evaluation_flow.ts

## 개요

AI가 생성한 UI JSON 출력물을 LLM 기반으로 평가하는 Genkit flow를 정의한다. 원본 사용자 요청, 생성된 출력, 평가에 사용할 모델 이름, JSON 스키마를 입력으로 받아 pass/fail 판정과 이슈 목록을 반환한다. 레이트 리미터를 통해 API 호출 빈도를 제어하며, 구조화된 JSON 출력 스키마를 통해 LLM의 응답을 파싱한다.

## 의존성

### 외부 패키지
- `genkit` — `z` (Zod 스키마)
- `js-yaml` — import되어 있으나 현재 핸들러 본문에서 직접 사용되지 않음

### 저장소 내부 모듈
- [`./ai`](./ai.ts.md)
- [`./rateLimiter`](./rateLimiter.ts.md)
- [`./logger`](./logger.ts.md)
- [`./types`](./types.ts.md) — `ProtocolSchemas` 타입
- `./models` (동적 import, `modelsToTest` 조회)

## Exports

| 이름 | 종류 | 설명 |
|------|------|------|
| `evaluationFlow` | Genkit Flow | UI 생성 결과를 LLM으로 평가하여 pass/fail 판정을 반환하는 flow |

## 상세 명세

### `EvalResultSchema` (내부 Zod 스키마, 핸들러 내부 선언)

핸들러 함수 내부에서 `z.object`로 정의된 LLM 응답 파싱용 스키마:
- `pass`: `z.boolean()` — UI가 요구사항을 충족하는지 여부
- `reason`: `z.string()` — 실패 원인 요약
- `issues`: `z.array(z.object({ issue: z.string(), severity: z.enum(['minor', 'significant', 'critical']) }))` — 개별 이슈 목록

### `evaluationFlow` (Genkit Flow, export)

`ai.defineFlow(config, handler)`로 정의된 flow.

**입력 스키마 (`z.object`):**
- `originalPrompt`: `string` — 원본 사용자 요청 텍스트
- `generatedOutput`: `string` — AI가 생성한 UI JSON 출력 (Markdown 코드블록 포함 가능)
- `evalModel`: `string` — 평가에 사용할 모델 이름
- `schemas`: `z.custom<ProtocolSchemas>()` — UI 프로토콜 JSON 스키마 맵

**출력 스키마 (`z.object`):**
- `pass`: `boolean`
- `reason`: `string`
- `issues`: 선택적, `{ issue: string, severity: 'minor'|'significant'|'critical' }[]`
- `evalPrompt`: 선택적 `string` — 디버깅용으로 실제 사용된 평가 프롬프트 반환

**핸들러 동작 단계:**

1. `schemas`의 모든 값을 `JSON.stringify(s, null, 2)`로 직렬화하여 줄바꿈으로 연결한 `schemaDefs` 문자열을 생성한다.

2. 평가 프롬프트(`evalPrompt`)를 구성한다. 프롬프트는 다음을 포함한다:
   - 사용자 요청 원문
   - 스키마 정의
   - 생성된 출력
   - 10개의 평가 지침: 컴포넌트 존재 여부, 계층 구조, 텍스트 내용 검증, 플랫 ID 참조 방식이 올바른지 확인(인라인 중첩 금지), 여분의 컴포넌트 허용, URL/레이블 관대한 평가 등
   - 3단계 심각도 정의: `minor`(미관적 차이), `significant`(사용성 저하), `critical`(컴포넌트 누락 또는 렌더링 불가)
   - JSON 스키마 형식으로 응답 구조 명시

3. 프롬프트 길이를 2.5로 나누어 `estimatedInputTokens`를 추정한다.

4. `./models`를 동적 import하여 `evalModel`에 해당하는 `evalModelConfig`를 찾는다. 없으면 `{ name: evalModel, model: null, requestsPerMinute: 60, tokensPerMinute: 100000 }` 기본값 사용.

5. `rateLimiter.acquirePermit(evalModelConfig, estimatedInputTokens)`를 await한다.

6. `ai.generate({ prompt: evalPrompt, model: evalModelConfig.model || evalModel, config: evalModelConfig.config, output: { schema: EvalResultSchema } })`를 호출하여 구조화된 JSON 응답을 파싱한다.

7. `response.output`이 없으면 `'No output from evaluation model'` 오류를 throw한다.

8. 성공 시 `{ pass, reason: reason || 'No reason provided', issues: issues || [], evalPrompt }`를 반환한다.

9. 예외 발생 시 `logger.error` 기록, `rateLimiter.reportError` 호출 후 예외를 re-throw한다(상위의 재시도 로직이 처리하도록).

## 동작 흐름

`Evaluator.runJob`에서 각 생성 결과에 대해 이 flow를 호출한다. flow는 프롬프트를 조립하고 레이트 리밋 허가를 받은 뒤 LLM에 구조화된 평가를 요청한다. 반환된 결과는 `Evaluator`에서 심각도 집계와 파일 저장에 사용된다. 예외 발생 시 `Evaluator`의 재시도 루프로 제어가 돌아간다.
