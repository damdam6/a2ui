# specification/v0_9/eval/src/evaluator.ts

## 개요

평가 파이프라인의 Phase 3를 담당하는 `Evaluator` 클래스를 정의한다. 스키마 검증을 통과한 결과들에 대해 `evaluationFlow`를 병렬로 호출하여 LLM 기반 품질 평가를 수행한다. 재시도 로직, 심각도 집계, 실패한 평가 결과의 파일 저장을 처리한다.

## 의존성

### 외부 패키지

- `fs` (Node.js 표준) — 파일 쓰기
- `path` (Node.js 표준) — 경로 구성
- `js-yaml` — `yaml.dump`로 YAML 직렬화

### 저장소 내부 모듈

- [`./evaluation_flow`](./evaluation_flow.ts.md) — `evaluationFlow`
- [`./types`](./types.ts.md) — `ValidatedResult`, `EvaluatedResult`, `IssueSeverity`
- [`./logger`](./logger.ts.md) — `logger` 인스턴스
- [`./rateLimiter`](./rateLimiter.ts.md) — `rateLimiter` 인스턴스

## Exports

- `Evaluator` — 클래스

## 상세 명세

### `Evaluator` 클래스

**생성자:**
- `schemas: any` — JSON 스키마 맵 (평가 프롬프트에 전달)
- `evalModel: string` — 평가에 사용할 모델 이름
- `outputDir?: string` — 실패 결과 저장 디렉토리 (선택)

**`run(results: ValidatedResult[]): Promise<EvaluatedResult[]>`**

Phase 3 평가를 실행하는 공개 메서드이다.

1. `validationErrors`가 비어 있고 `components`가 존재하는 항목을 `passedResults`로 분류한다. 나머지는 건너뛴다.
2. 스킵 항목(`validationErrors`가 있는 경우)을 `evaluatedResults`에 즉시 추가한다. 이 때 `evaluationResult`는 `pass: false`, `reason: 'Schema validation failure'`, `severity: 'criticalSchema'`로 설정된다. `components`가 없지만 오류도 없는 항목은 평가 결과 없이 그대로 추가된다.
3. 평가할 항목이 없으면 즉시 반환한다.
4. `setInterval(1000)`으로 진행률 모니터를 시작한다. 매 초 `process.stderr`에 `[Phase 3] Progress: <pct>% | Completed: ... | In Progress: ... | Queued: ... | Failed: ...` 형태로 출력한다. `rateLimiter.waitingCount`를 큐 대기 수로 사용한다.
5. `passedResults`의 각 항목에 대해 `runJob(result)`를 호출하는 Promise를 생성하여 병렬로(`Promise.all`) 실행한다.
6. 각 Promise 완료 시 `evaluationResult`가 있으면 `completedCount`를, 없으면 `failedCount`를 증가시킨다.
7. 모든 Promise 완료 후 인터벌을 정리하고 `\n`을 출력한다.
8. 최종 `evaluatedResults` 배열을 반환한다.

**`private runJob(result: ValidatedResult): Promise<EvaluatedResult>`**

단일 검증 결과에 대한 평가 작업을 실행하는 비공개 메서드이다.

1. `maxEvalRetries = 3` 회 재시도 루프를 실행한다.
2. 각 시도에서 `evaluationFlow({ originalPrompt, generatedOutput, evalModel, schemas })`를 호출한다.
3. 성공 시 루프를 `break`한다.
4. 예외 발생 시:
   - 마지막 시도(`evalRetry === 2`)이면 `evaluationResult`를 `pass: false, reason: 'Evaluation flow failed: ...'`로 설정하고 루프를 종료한다.
   - 그렇지 않으면 `1000 * 2^evalRetry` ms 지수 백오프 후 재시도한다.
5. 평가 결과가 실패(`!pass`)이고 `issues`가 있으면 심각도를 집계한다. `'critical'` > `'significant'` > `'minor'` 우선순위로 `overallSeverity`를 결정한다.
6. `outputDir`가 설정되어 있고 평가 결과가 있으면 `saveEvaluation`을 호출한다.
7. `{ ...result, evaluationResult: { ...evaluationResult, overallSeverity } }`를 반환한다. 평가 결과가 없으면 `evaluationResult: undefined`로 반환한다.

**`private saveEvaluation(result, evaluationResult, overallSeverity?)`**

평가 실패 결과를 파일로 저장하는 비공개 메서드이다.

1. `outputDir`가 없거나 `evaluationResult.pass === true`이면 즉시 반환한다.
2. 저장 경로: `<outputDir>/output-<modelName(특수문자를 _로)>/details/<promptName>.<runNumber>.failed.yaml`
3. `yaml.dump({ ...evaluationResult, overallSeverity })`를 YAML 파일로 저장한다.
4. `evaluationResult.evalPrompt`가 있으면 `<promptName>.<runNumber>.eval_prompt.txt` 파일로 추가 저장한다.

## 동작 흐름

`index.ts`의 `main`에서 `validator.run()` 결과를 받아 → 스킵 항목 선별 → 병렬 LLM 평가 (재시도 포함) → 심각도 집계 → 실패 결과 파일 저장 → `EvaluatedResult[]` 반환.
