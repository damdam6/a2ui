# specification/v1_0/eval/src/evaluator.ts

## 개요

유효성 검증을 통과한 생성 결과를 LLM 기반으로 일괄 평가하는 `Evaluator` 클래스를 제공한다. 유효성 검증 실패 항목을 즉시 처리하고, 나머지 항목은 `evaluationFlow`를 병렬로 호출하여 평가한다. 최대 3회 지수 백오프 재시도를 지원하며, 평가 실패 결과를 YAML 파일로 저장할 수 있다.

## 의존성

### 외부 패키지
- `fs` (Node.js 내장)
- `path` (Node.js 내장)
- `js-yaml` — YAML 직렬화

### 저장소 내부 모듈
- [`./evaluation_flow`](./evaluation_flow.ts.md) — `evaluationFlow`
- [`./types`](./types.ts.md) — `ValidatedResult`, `EvaluatedResult`, `ProtocolSchemas`, `IssueSeverity`
- [`./logger`](./logger.ts.md)
- [`./rateLimiter`](./rateLimiter.ts.md)

## Exports

| 이름 | 종류 | 설명 |
|------|------|------|
| `Evaluator` | 클래스 | 병렬 LLM 평가를 수행하고 결과를 집계하는 클래스 |

## 상세 명세

### `Evaluator` 클래스

#### 생성자

```
constructor(
  private schemas: ProtocolSchemas,
  private evalModel: string,
  private outputDir?: string
)
```

- `schemas`: UI 프로토콜 JSON 스키마 맵, `evaluationFlow`에 그대로 전달
- `evalModel`: 평가에 사용할 모델 이름 문자열
- `outputDir`: 선택적. 평가 실패 결과를 저장할 기본 출력 디렉토리

#### `run(results: ValidatedResult[]): Promise<EvaluatedResult[]>` (public)

전체 평가 워크플로를 실행한다.

1. `validationErrors.length === 0`이고 `components`가 존재하는 항목만 `passedResults`로 필터링한다. 나머지(`skippedCount`)는 유효성 검증 실패로 건너뛴다.

2. 유효성 검증 실패 항목은 `evaluatedResults`에 즉시 추가한다:
   - `validationErrors.length > 0`인 경우: `evaluationResult.pass = false`, `reason = 'Schema validation failure'`, `issues = [{ issue: validationErrors.join('\n'), severity: 'criticalSchema' }]`, `overallSeverity = 'criticalSchema'`
   - `components`가 없는 경우: `evaluationResult` 없이 원본 결과만 추가

3. `totalJobs === 0`이면 즉시 `evaluatedResults` 반환.

4. 1초 간격의 `setInterval`로 진행 상황을 `process.stderr`에 출력한다. 형식: `[Phase 3] Progress: <pct>% | Completed: <n> | In Progress: <n> | Queued: <n> | Failed: <n>`. 대기 중인 수는 `rateLimiter.waitingCount`에서 읽는다.

5. `passedResults`의 각 항목에 대해 `this.runJob(result)`를 호출하는 Promise를 생성하여 `Promise.all`로 병렬 실행한다. 각 Promise 완료 시 `completedCount` 또는 `failedCount`를 증가시키고 결과를 `evaluatedResults`에 push한다.

6. 모든 Promise 완료 후 인터벌 클리어, 개행 출력, 완료 로그 기록 후 `evaluatedResults` 반환.

#### `runJob(result: ValidatedResult): Promise<EvaluatedResult>` (private)

단일 항목에 대한 평가를 최대 3회(`maxEvalRetries = 3`) 재시도한다.

1. `for` 루프로 최대 3회 반복하며 `evaluationFlow({ originalPrompt: result.prompt.promptText, generatedOutput: result.rawText || '', evalModel: this.evalModel, schemas: this.schemas })`를 await 호출한다.

2. 성공하면 `break`으로 루프를 탈출한다.

3. 예외 발생 시:
   - 마지막 시도(`evalRetry === maxEvalRetries - 1`)이면 경고를 기록하고 `{ pass: false, reason: 'Evaluation flow failed: <message>' }`를 설정한다.
   - 아니면 `1000 * 2^evalRetry` ms 지수 백오프 대기 후 재시도한다.

4. 평가 결과가 실패(`!evaluationResult.pass`)이고 `issues`가 있으면 가장 높은 심각도를 `overallSeverity`로 결정한다: `critical` > `significant` > `minor` 순으로 우선순위 적용.

5. `outputDir`가 설정되어 있고 평가 결과가 있으면 `saveEvaluation`을 호출한다.

6. `{ ...result, evaluationResult: evaluationResult ? { ...evaluationResult, overallSeverity } : undefined }`를 반환한다.

#### `saveEvaluation(result, evaluationResult, overallSeverity)` (private)

평가 실패(`evaluationResult.pass === false`)인 경우에만 파일을 저장한다.

- 저장 경로: `<outputDir>/output-<modelName>/details/<promptName>.<runNumber>.failed.yaml`
  - `modelName`의 `/`, `:` 문자는 `_`로 치환
  - 파일 내용: `yaml.dump({ ...evaluationResult, overallSeverity })`
- `evaluationResult.evalPrompt`가 존재하면 동일 디렉토리에 `<promptName>.<runNumber>.eval_prompt.txt`로 저장.
- 주의: `detailsDir`의 존재 여부를 확인하지 않으므로 디렉토리는 사전에 생성되어 있어야 한다(Generator가 `saveArtifacts`에서 생성).

## 동작 흐름

`index.ts`의 `main`에서 Phase 3로 호출된다. `run(validatedResults)`를 받아 유효성 검증 실패는 즉시 처리하고, 통과 항목은 `Promise.all`로 병렬 평가를 수행한다. 각 평가는 내부적으로 레이트 리미터를 거쳐 `evaluationFlow`를 호출하며, 최대 3회 재시도한다. 결과는 `EvaluatedResult[]`로 반환되어 Phase 4 분석과 최종 요약에 사용된다.
