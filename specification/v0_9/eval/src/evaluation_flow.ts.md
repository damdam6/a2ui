# specification/v0_9/eval/src/evaluation_flow.ts

## 개요

생성된 UI JSON 출력물이 사용자 요청 및 프로토콜 스키마를 충족하는지 LLM 기반으로 평가하는 Genkit 플로우를 정의한다. 평가 결과로 합격 여부(`pass`), 실패 사유(`reason`), 개별 이슈 목록(심각도 포함)을 반환한다. 레이트 리미터를 통해 API 호출 속도를 제어하며, 오류 발생 시 상위 재시도 로직에서 처리할 수 있도록 예외를 다시 던진다.

## 의존성

### 외부 패키지

- `genkit` — `z` (Zod 스키마)
- `js-yaml` — `yaml` (import만 되어 있고 이 파일에서 직접 사용되지 않음)

### 저장소 내부 모듈

- [`./ai`](./ai.ts.md) — `ai` 인스턴스
- [`./rateLimiter`](./rateLimiter.ts.md) — `rateLimiter` 인스턴스
- [`./logger`](./logger.ts.md) — `logger` 인스턴스
- `./models` — `modelsToTest` (동적 import)

## Exports

- `evaluationFlow` — Genkit 플로우 객체 (상수)

## 상세 명세

### `evaluationFlow` (Genkit 플로우)

`ai.defineFlow(config, handler)`로 정의된 플로우이다.

**플로우 이름:** `'evaluationFlow'`

**inputSchema:**
- `originalPrompt`: `z.string()` — 원본 사용자 요청 텍스트
- `generatedOutput`: `z.string()` — 생성된 UI JSON (Markdown 코드 블록 형태의 JSONL)
- `evalModel`: `z.string()` — 평가에 사용할 모델 이름
- `schemas`: `z.any()` — JSON 스키마 객체 맵

**outputSchema:**
- `pass`: `z.boolean()` — 평가 합격 여부
- `reason`: `z.string()` — 실패 사유 요약
- `issues`: (선택) 이슈 객체 배열. 각 이슈는 `issue: z.string()` 및 `severity: z.enum(['minor', 'significant', 'critical'])`를 가진다.
- `evalPrompt`: (선택) `z.string()` — 실제 사용된 평가 프롬프트 (디버깅용)

**내부 스키마 `EvalResultSchema`:**

핸들러 내부에서 정의되는 Zod 스키마로, `pass`, `reason`, `issues` 필드를 가지며 `ai.generate`의 구조화 출력 스키마로 사용된다.

**핸들러 동작:**

1. `schemas` 객체의 모든 값을 `JSON.stringify(..., null, 2)`하여 줄바꿈으로 이어 붙인 `schemaDefs` 문자열을 만든다.
2. `evalPrompt` 문자열을 구성한다. 이 프롬프트는:
   - 역할: `"You are an expert QA evaluator for a UI generation system."`
   - 원본 요청, 기대 스키마(`schemaDefs`), 생성 출력물 포함
   - 평가 지침 10개 항목 포함. 주요 항목:
     - 모든 요청된 컴포넌트가 존재하고 계층 구조가 일치하는지 확인
     - 자식 컴포넌트는 인라인 객체가 아닌 문자열 ID로 참조하는 것이 올바른 방식임을 명시
     - URL, 레이블 텍스트에 대해서는 관대하게 평가 허용
     - 명시되지 않은 추가 컴포넌트/속성은 허용
   - 심각도 정의: `minor` (단순 외형), `significant` (사용성 저하), `critical` (컴포넌트 누락 또는 렌더링 불가)
   - JSON Schema 형식의 기대 출력 구조를 프롬프트 내에 포함
3. `evalPrompt.length / 2.5`를 올림하여 `estimatedInputTokens`를 계산한다.
4. `./models`를 동적 import하여 `evalModel`에 해당하는 설정을 찾는다. 없으면 `requestsPerMinute: 60`, `tokensPerMinute: 100000`인 기본 폴백 객체를 생성한다.
5. `rateLimiter.acquirePermit(evalModelConfig, estimatedInputTokens)`로 요청 허가를 기다린다.
6. `ai.generate({ prompt, model, config, output: { schema: EvalResultSchema } })`를 호출하여 구조화된 JSON 결과를 받는다.
7. `response.output`이 없으면 `'No output from evaluation model'` 오류를 던진다.
8. 성공 시 `{ pass, reason, issues, evalPrompt }`를 반환한다. `reason`이 없으면 `'No reason provided'`로, `issues`가 없으면 빈 배열로 대체한다.
9. 예외 발생 시 `logger.error`로 기록하고 `rateLimiter.reportError`를 호출한 뒤 예외를 **재발생**시킨다 (상위 `Evaluator.runJob`에서 재시도 처리).

## 동작 흐름

`Evaluator.runJob`에서 단일 검증 결과에 대해 호출됨 → 스키마 문자열화 → 평가 프롬프트 구성 → 레이트 리미터 허가 → LLM 구조화 출력 요청 → 합격/불합격 결과 반환 또는 예외 재발생.
