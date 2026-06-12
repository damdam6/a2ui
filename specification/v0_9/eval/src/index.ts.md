# specification/v0_9/eval/src/index.ts

## 개요

평가 파이프라인의 CLI 진입점이다. `yargs`로 커맨드라인 인수를 파싱하고, 스키마 로딩 → Phase 1(생성) → Phase 2(검증) → Phase 3(LLM 평가) → Phase 4(실패 분석) → 요약 출력/저장의 전체 평가 흐름을 순차적으로 실행한다.

## 의존성

### 외부 패키지

- `fs` (Node.js 표준) — 파일 읽기/쓰기/삭제
- `path` (Node.js 표준) — 경로 구성
- `yargs` / `yargs/helpers` — CLI 인수 파싱

### 저장소 내부 모듈

- [`./logger`](./logger.ts.md) — `logger`, `setupLogger`
- [`./models`](./models.ts.md) — `modelsToTest`
- [`./prompts`](./prompts.ts.md) — `prompts`, `TestPrompt`
- [`./generator`](./generator.ts.md) — `Generator`
- [`./validator`](./validator.ts.md) — `Validator`
- [`./evaluator`](./evaluator.ts.md) — `Evaluator`
- [`./types`](./types.ts.md) — `EvaluatedResult`
- [`./analysis_flow`](./analysis_flow.ts.md) — `analysisFlow`

## Exports

없음 (CLI 스크립트로 직접 실행)

## 상세 명세

### `schemaFiles` (상수, `string[]`)

로드할 JSON 스키마 파일의 상대 경로 목록이다. `__dirname` 기준으로 다음 세 파일을 참조한다:
- `'../../json/common_types.json'`
- `'../../catalogs/basic/catalog.json'`
- `'../../json/server_to_client.json'`

### `loadSchemas(): Record<string, any>`

`schemaFiles`를 읽어 스키마 맵을 반환하는 함수이다.

1. 각 파일을 `fs.readFileSync`로 읽어 JSON 파싱 후, `'../../'`을 제거한 경로를 키로 하여 `schemas` 객체에 저장한다.
2. `'catalogs/basic/catalog.json'`이 로드된 경우, 그 복사본의 `$id` 필드 끝에 있는 `'catalogs/basic/catalog.json'`을 `'catalog.json'`으로 교체하여 `schemas['catalog.json']`에 별칭으로 추가한다. 이는 `server_to_client.json`의 참조와 일치시키기 위함이다.
3. 완성된 `schemas` 맵을 반환한다.

### `generateSummary(results: EvaluatedResult[], analysisResults: Record<string, string>): string`

평가 결과 전체를 Markdown 테이블과 통계로 정리한 요약 문자열을 생성하는 함수이다.

1. 컬럼 너비 상수를 정의한다: `promptNameWidth=40`, `latencyWidth=20`, `failedRunsWidth=15`, `severityWidth=15`.
2. `results`를 `modelName`으로 그룹화한다.
3. 각 모델에 대해:
   - `## Model: <modelName>` 헤더 추가
   - `Prompt Name`, `Avg Latency (ms)`, `Schema Fail`, `Eval Fail`, `Minor`, `Significant`, `Critical` 컬럼을 가진 Markdown 테이블 생성
   - 프롬프트별로 정렬하여 각 행에 통계를 채운다: 평균 레이턴시, 스키마 실패 비율, 평가 실패 비율, 심각도별 이슈 수
   - 모델 전체 스키마 성공률 및 전체 성공률 계산 및 출력
   - `analysisResults[modelName]`이 있으면 `### Failure Analysis` 섹션 추가
4. `## Overall Summary` 섹션 추가:
   - 전체 도구 오류 수, 전체 실패 수 및 성공률
   - 심각도별 이슈 합계 (minor, significant, critical, criticalSchema)
   - 전체 평균/중앙값 레이턴시 (ms)
   - 실패가 있는 모델 목록
5. 완성된 Markdown 문자열을 반환한다.

### `main(): Promise<void>`

CLI 진입점 함수이다. `require.main === module` 조건 하에 실행된다.

**CLI 인수 (yargs 옵션):**
- `--log-level`: `string`, 기본값 `'info'`, 선택지 `['debug', 'info', 'warn', 'error']`
- `--results`: `string` 또는 `true`. 미지정 시 `true` (기본값), 지정 시 해당 문자열이 결과 기반 디렉토리로 사용됨
- `--runs-per-prompt`: `number`, 기본값 `1`
- `--model`: `string[]`, 기본값 `[]`, `modelsToTest`의 이름들 중에서 선택
- `--prompt`: `string[]`, 프롬프트 이름 접두사로 필터링
- `--eval-model`: `string`, 기본값 `'gemini-2.5-flash'`, `modelsToTest`의 이름들 중에서 선택
- `--clean-results`: `boolean`, 기본값 `false`

**실행 흐름:**

1. **모델 필터링:** `--model`이 지정된 경우 `modelsToTest`에서 이름이 일치하는 모델만 선택한다. 결과가 비어 있으면 오류 로그 후 `process.exit(1)`.
2. **프롬프트 필터링:** `--prompt`가 지정된 경우 `prompts`에서 이름이 해당 접두사로 시작하는 것만 선택한다. 결과가 비어 있으면 오류 로그 후 `process.exit(1)`.
3. **출력 디렉토리 결정:** `--results`가 문자열이면 그 값을, `true`이면 `'results'`를 `resultsBaseDir`로 사용한다.
4. **결과 정리:** `--clean-results`가 `true`이고 `resultsBaseDir`가 존재하면 `fs.rmSync(..., { recursive: true, force: true })`로 삭제한다.
5. **로거 설정:** 단일 모델인 경우 해당 모델의 출력 디렉토리에 파일 로거를 설정한다. 복수 모델이거나 디렉토리 미설정 시 `setupLogger(undefined, logLevel)`.
6. **스키마 로드:** `loadSchemas()` 호출.
7. **카탈로그 규칙 로드:** `__dirname/../../json/basic_catalog_rules.txt` 파일이 존재하면 읽고, 없으면 경고 로그.
8. **Phase 1 (생성):** `new Generator(schemas, resultsBaseDir, catalogRules).run(filteredPrompts, filteredModels, runsPerPrompt)` 호출.
9. **Phase 2 (검증):** `new Validator(schemas, resultsBaseDir).run(generatedResults)` 호출.
10. **Phase 3 (평가):** `new Evaluator(schemas, evalModel, resultsBaseDir).run(validatedResults)` 호출.
11. **Phase 4 (실패 분석):** 평가 결과를 모델별로 그룹화한 뒤, 각 모델의 실패 항목(도구 오류, 검증 실패, 평가 실패)을 분류하여 `analysisFlow`에 전달한다. 실패 항목이 없는 모델은 건너뛴다.
12. **요약 생성 및 출력:** `generateSummary(evaluatedResults, analysisResults)` 호출 후 `logger.info(summary)`.
13. **요약 파일 저장:** `resultsBaseDir`가 설정된 경우, 각 모델의 출력 디렉토리에 `summary.md` 파일을 저장한다.

## 동작 흐름

`node index.js [options]` 실행 → CLI 파싱 → 스키마/규칙 로드 → 4단계 파이프라인 순차 실행 → Markdown 요약 출력 및 파일 저장.
