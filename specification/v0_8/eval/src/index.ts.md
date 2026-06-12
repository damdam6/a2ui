# specification/v0_8/eval/src/index.ts

## 개요

이 파일은 평가 시스템의 주 실행 진입점이다. CLI 인자를 파싱하여 테스트할 모델과 프롬프트를 필터링하고, 모든 조합에 대해 `componentGeneratorFlow`를 병렬로 실행한 뒤 결과를 검증하고 마크다운 요약 리포트를 출력(및 선택적으로 파일로 저장)한다. `require.main === module` 가드로 직접 실행 시에만 `main()`이 호출된다.

## 의존성

### 외부 패키지
- `fs` (Node.js 내장) — 파일 읽기/쓰기, 디렉토리 생성
- `path` (Node.js 내장) — 파일 경로 조합
- `yargs` — CLI 인자 파싱
- `yargs/helpers` — `hideBin` 유틸리티

### 저장소 내부 모듈
- [`./flows`](./flows.ts.md) — `componentGeneratorFlow`, `ai`
- [`./models`](./models.ts.md) — `modelsToTest`
- [`./prompts`](./prompts.ts.md) — `prompts`, `TestPrompt`
- `./validator` — `validateSchema` (목록 외 파일이나 import됨)

## Exports

없음 (스크립트 진입점 파일)

## 상세 명세

### `InferenceResult` (interface, 비공개)

단일 평가 실행 결과를 담는 내부 타입이다.

- `modelName: string` — 모델 이름.
- `prompt: TestPrompt` — 사용된 프롬프트 객체.
- `component: any` — 생성된 컴포넌트 JSON. 오류 시 `null`.
- `error: any` — 발생한 오류. 정상 시 `null`.
- `latency: number` — 생성 소요 시간(밀리초).
- `validationResults: string[]` — 검증 실패 메시지 배열. 성공 시 빈 배열.
- `runNumber: number` — 동일 프롬프트의 반복 실행 번호(1부터 시작).

### `generateSummary(resultsByModel: Record<string, InferenceResult[]>, results: InferenceResult[]): string`

마크다운 형식의 요약 문자열을 생성하여 반환한다.

**컬럼 너비 상수**:
- `promptNameWidth = 40`, `latencyWidth = 20`, `failedRunsWidth = 15`, `toolErrorRunsWidth = 20`

**동작 단계**:
1. `# Evaluation Summary`로 시작.
2. `resultsByModel`의 각 모델명에 대해 `## Model: ${modelName}` 섹션을 생성한다.
3. 각 모델의 결과를 프롬프트 이름별로 `promptsInModel` 맵으로 그룹화한다.
4. 프롬프트별 통계를 계산한다: 전체 런 수, 오류 런 수(`r.error` truthy), 실패 런 수(오류 또는 `validationResults.length > 0`), 평균 지연 시간.
5. 각 프롬프트 행을 마크다운 테이블 형식으로 추가한다. 실패가 없으면 해당 셀은 빈 문자열.
6. 모델별 총 실패 런 수(`**Total failed runs:**`)를 추가.
7. `---` 구분선 후 `## Overall Summary` 섹션 추가:
   - 전체 런 수 대비 도구 오류 런 수.
   - 전체 런 수 대비 임의 실패(오류 또는 검증 실패) 런 수.
   - 지연 시간 배열을 오름차순 정렬하여 평균(mean)과 중앙값(median) 계산. 중앙값은 짝수 개면 중간 두 값의 평균, 홀수 개면 중간 값.
   - 최소 하나의 실패가 있는 모델 이름 목록(`Set`으로 중복 제거).

### `main()` (비동기 함수)

CLI 파싱 및 평가 실행을 담당한다.

**CLI 옵션**:
- `--verbose` / `-v` (boolean, default `false`) — 검증 실패 시 생성된 JSON 전체 출력 여부.
- `--keep` (string, 선택) — 결과 파일을 저장할 디렉토리 경로. 값 없이 플래그만 주면 `true`로 coerce되어 `a2ui-eval-` 접두사로 임시 디렉토리를 생성한다.
- `--runs-per-prompt` (number, default `1`) — 프롬프트당 반복 실행 횟수.
- `--model` (string[], default `[]`, choices: `modelsToTest`의 name 목록) — 특정 모델만 선택.
- `--prompt` (string, 선택) — 이름 접두사로 프롬프트 필터링.

**실행 흐름**:
1. CLI 인자를 파싱하여 `verbose`, `keep`, `runsPerPrompt` 등의 변수를 초기화.
2. `--keep`이 있으면 출력 디렉토리 `outputDir`를 결정하고 없으면 생성(`recursive: true`).
3. `--model`이 제공되면 `modelsToTest`를 필터링하여 `filteredModels`를 만든다. 일치하는 모델이 없으면 `process.exit(1)`.
4. `--prompt`가 제공되면 `prompts`를 이름 접두사로 필터링하여 `filteredPrompts`를 만든다. 일치하는 프롬프트가 없으면 `process.exit(1)`.
5. 이중 루프(`filteredPrompts` × `filteredModels` × `runsPerPrompt`)를 순회하며 각 조합에 대해:
   - 프롬프트의 `schemaPath`에서 JSON 스키마를 `fs.readFileSync`로 읽어 `JSON.parse`.
   - 모델 이름에서 `/`와 `:`를 `_`로 치환한 디렉토리 이름을 만들고 `outputDir`가 있으면 모델별 하위 디렉토리를 생성.
   - `componentGeneratorFlow()` 호출 Promise를 `generationPromises` 배열에 추가.
   - 성공 콜백: `outputDir`가 있으면 입력 텍스트(`.input.txt`)와 출력 JSON(`.output.json`)을 저장. `validateSchema()`를 호출하여 검증 결과를 얻는다. `InferenceResult` 객체를 구성하여 반환.
   - 실패 콜백: `outputDir`가 있으면 입력 텍스트와 오류 JSON(`.error.json`)을 저장. 오류를 포함한 `InferenceResult` 객체를 반환.
6. `Promise.all(generationPromises)`로 모든 생성을 병렬 대기.
7. 결과를 `resultsByModel` 맵으로 모델별 그룹화.
8. 결과를 콘솔에 출력: 오류가 있거나 검증 실패가 있거나 `verbose`이고 컴포넌트가 있는 경우에만 상세 출력.
9. `generateSummary()`를 호출하여 요약 문자열을 생성하고 콘솔에 출력. `outputDir`가 있으면 `summary.md`로 저장.

### 진입점 가드

`if (require.main === module)` 조건으로 직접 실행 시에만 `main().catch(console.error)`를 호출한다.

## 동작 흐름

스크립트 실행 → CLI 파싱 → 모델/프롬프트 필터링 → 스키마 파일 로드 → 병렬 LLM 생성 실행(Promise.all) → 각 결과에 대해 검증 수행 → 결과 그룹화 → 콘솔 출력 및 선택적 파일 저장 → 마크다운 요약 출력.

병렬 실행이 핵심 특징으로, 모든 모델·프롬프트 조합을 동시에 실행하여 전체 평가 시간을 최소화한다.
