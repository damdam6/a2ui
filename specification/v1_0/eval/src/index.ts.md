# specification/v1_0/eval/src/index.ts

## 개요

평가 파이프라인의 메인 진입점이다. CLI 인수를 파싱하여 모델/프롬프트 필터링, 출력 디렉토리 설정, 로그 레벨 구성을 수행한 뒤, Generation → Validation → Evaluation → Failure Analysis → Summary의 4단계 파이프라인을 순서대로 실행하고 Markdown 요약 보고서를 출력 및 저장한다.

## 의존성

### 외부 패키지
- `fs` (Node.js 내장)
- `path` (Node.js 내장)
- `yargs` — CLI 인수 파싱
- `yargs/helpers` — `hideBin`

### 저장소 내부 모듈
- [`./logger`](./logger.ts.md) — `logger`, `setupLogger`
- [`./models`](./models.ts.md) — `modelsToTest`
- [`./prompts`](./prompts.ts.md) — `prompts`, `TestPrompt`
- [`./generator`](./generator.ts.md) — `Generator`
- [`./validator`](./validator.ts.md) — `Validator`
- [`./evaluator`](./evaluator.ts.md) — `Evaluator`
- [`./types`](./types.ts.md) — `EvaluatedResult`, `ProtocolSchemas`
- [`./analysis_flow`](./analysis_flow.ts.md) — `analysisFlow`

## Exports

없음 (스크립트로 직접 실행될 때만 `main()`이 호출됨)

## 상세 명세

### `schemaFiles` (const)

로드할 스키마 파일의 상대 경로 배열. `__dirname` 기준 값:
- `'../../json/common_types.json'`
- `'../../catalogs/basic/catalog.json'`
- `'../../json/server_to_client.json'`

### `loadSchemas(): ProtocolSchemas` (함수)

1. `schemaFiles`를 순회하며 각 파일을 `fs.readFileSync(..., 'utf-8')`로 읽고 `JSON.parse`한다.
2. 키는 `file.replace('../../', '')`로 생성 (예: `'json/common_types.json'`).
3. `'catalogs/basic/catalog.json'` 항목이 있으면 딥 클론 후 `$id` 필드의 `catalogs/basic/catalog.json` 부분을 `catalog.json`으로 치환하여 `schemas['catalog.json']` 별칭으로 추가한다. 이는 `server_to_client.json`이 `$ref`에서 `catalog.json`을 참조하기 때문이다.
4. `ProtocolSchemas` 객체를 반환한다.

### `generateSummary(results, analysisResults): string` (함수)

- **매개변수:**
  - `results: EvaluatedResult[]`
  - `analysisResults: Record<string, string>` — 모델명 → 분석 텍스트 맵

**동작:**

1. 결과를 `resultsByModel`(모델명 → 결과 배열)로 그룹화한다.

2. 각 모델에 대해 Markdown 표를 생성한다. 컬럼 너비:
   - `promptNameWidth = 40`, `latencyWidth = 20`, `failedRunsWidth = 15`, `severityWidth = 15`
   - 컬럼 헤더: `Prompt Name`, `Avg Latency (ms)`, `Schema Fail`, `Eval Fail`, `Minor`, `Significant`, `Critical`

3. 프롬프트별 집계: 알파벳 정렬된 프롬프트 이름으로 반복하며 각각의 총 실행 수, 스키마 실패 수(`r.error || r.validationErrors.length > 0`), 평가 실패 수(`r.evaluationResult && !r.evaluationResult.pass`), 평균 레이턴시, 심각도별 이슈 수(minor/significant/critical)를 계산하고 테이블 행으로 추가한다. 값이 0이면 빈 문자열로 표시.

4. 모델 푸터에 스키마 성공 비율(`schemaSuccessfulRuns / total * 100`)과 전체 성공 비율(`successfulRuns / total * 100`)을 소수점 1자리로 출력.

5. `analysisResults[modelName]`이 있으면 `### Failure Analysis` 섹션으로 추가.

6. 전체 요약(`## Overall Summary`) 섹션:
   - 총 도구 오류 수
   - 실패(도구 오류, 유효성 검증, 평가 실패) 비율과 성공 비율
   - 심각도별 전체 집계(minor/significant/critical/criticalSchema)
   - 평균 레이턴시 및 중앙값 레이턴시 (ms): 배열을 정렬 후 홀수 개이면 중앙 값, 짝수 개이면 중앙 두 값의 평균
   - 하나 이상 실패한 모델 이름 목록 (중복 제거)

7. 완성된 Markdown 문자열을 반환한다.

### `main(): Promise<void>` (async 함수)

**CLI 옵션 정의 (`yargs`):**

| 옵션 | 타입 | 기본값 | 설명 |
|------|------|--------|------|
| `--log-level` | string | `'info'` | 로그 레벨. 선택지: `debug`, `info`, `warn`, `error` |
| `--results` | string\|boolean | `true` | 출력 디렉토리. 미지정 시 `true`(→ `'results'`), 문자열 지정 시 해당 경로 |
| `--runs-per-prompt` | number | `1` | 프롬프트당 실행 횟수 |
| `--model` | string[] | `[]` | 정확한 이름으로 모델 필터링. `modelsToTest`의 이름 중에서 선택 |
| `--prompt` | string[] | - | 이름 접두사로 프롬프트 필터링 |
| `--eval-model` | string | `'gemini-2.5-flash'` | 평가에 사용할 모델. `modelsToTest`의 이름 중에서 선택 |
| `--clean-results` | boolean | `false` | 출력 디렉토리 시작 전 삭제 여부 |

**실행 단계:**

1. **모델 필터링:** `--model`이 지정된 경우 `modelsToTest`에서 이름이 정확히 일치하는 모델만 선택. 결과가 비어 있으면 오류 로그 후 `process.exit(1)`.

2. **프롬프트 필터링:** `--prompt`가 지정된 경우 `prompts`에서 `p.name.startsWith(prefix)`인 것만 선택. 결과가 비어 있으면 오류 로그 후 `process.exit(1)`.

3. **출력 디렉토리 결정:** `resultsArg`가 문자열이면 그대로 사용, `true`이면 `'results'` 사용.

4. **정리:** `--clean-results`가 `true`이고 디렉토리가 존재하면 `fs.rmSync(dir, { recursive: true, force: true })`로 삭제.

5. **로거 설정:** 단일 모델 테스트 시 `setupLogger(path.join(resultsBaseDir, modelDirName), logLevel)` 호출로 파일 로깅 활성화. 복수 모델이면 `setupLogger(undefined, logLevel)`.

6. **스키마 로드:** `loadSchemas()`로 프로토콜 스키마 맵 생성. `schemas['catalogs/basic/catalog.json']?.instructions`에서 카탈로그 규칙을 읽는다.

7. **Phase 1 — Generation:** `new Generator(schemas, resultsBaseDir, catalogRules).run(filteredPrompts, filteredModels, runsPerPrompt)`.

8. **Phase 2 — Validation:** `new Validator(schemas, resultsBaseDir).run(generatedResults)`.

9. **Phase 3 — Evaluation:** `new Evaluator(schemas, evalModel, resultsBaseDir).run(validatedResults)`.

10. **Phase 4 — Failure Analysis:** `evaluatedResults`를 모델별로 그룹화. 각 모델에서 실패(도구 오류, 유효성 검증 실패, 평가 실패) 항목을 추출하여 `{ promptName, runNumber, failureType, reason, issues }` 형태로 변환한 뒤, 실패가 있을 경우 `analysisFlow({ modelName, failures, numRuns, evalModel })`를 호출하여 분석 텍스트를 얻는다. 분석 호출 실패 시 `'Failed to run analysis.'` 문자열을 기록.

11. **요약 생성 및 저장:** `generateSummary(evaluatedResults, analysisResults)`를 호출하여 Markdown 요약을 생성하고 `logger.info`로 출력. `resultsBaseDir`가 있으면 각 모델 디렉토리에 `summary.md`로 저장.

### 진입점 guard

```js
if (require.main === module) {
  main().catch(console.error);
}
```

스크립트로 직접 실행될 때만 `main()`을 호출한다.

## 동작 흐름

`node index.js [options]` 형태로 실행된다. yargs로 CLI를 파싱한 뒤 필터링, 정리, 로거 설정을 거쳐 4단계 파이프라인을 순차적으로 실행한다. 각 단계(Generator, Validator, Evaluator)는 내부적으로 병렬 처리를 사용한다. Phase 4의 분석과 최종 요약은 동기적으로 처리되어 파일로 저장된다.
