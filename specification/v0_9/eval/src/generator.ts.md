# specification/v0_9/eval/src/generator.ts

## 개요

평가 파이프라인의 Phase 1을 담당하는 `Generator` 클래스를 정의한다. 테스트 프롬프트와 모델 설정의 모든 조합에 대해 `componentGeneratorFlow`를 병렬로 호출하여 UI JSON을 생성하고, 성공 시 아티팩트를 파일로 저장하며, 오류 발생 시 1회 재시도 후 오류 결과를 반환한다.

## 의존성

### 외부 패키지

- `fs` (Node.js 표준) — 파일 읽기/쓰기
- `path` (Node.js 표준) — 경로 구성

### 저장소 내부 모듈

- [`./generation_flow`](./generation_flow.ts.md) — `componentGeneratorFlow`
- [`./models`](./models.ts.md) — `ModelConfiguration` 타입
- [`./prompts`](./prompts.ts.md) — `TestPrompt` 타입
- [`./types`](./types.ts.md) — `GeneratedResult` 타입
- [`./utils`](./utils.ts.md) — `extractJsonFromMarkdown`
- [`./rateLimiter`](./rateLimiter.ts.md) — `rateLimiter` 인스턴스
- [`./logger`](./logger.ts.md) — `logger` 인스턴스

## Exports

- `Generator` — 클래스

## 상세 명세

### `Generator` 클래스

**생성자:**
- `schemas: any` — JSON 스키마 맵 (플로우에 전달)
- `outputDir?: string` — 아티팩트 저장 디렉토리 (선택)
- `catalogRules?: string` — 카탈로그별 추가 지시사항 (선택)

**`run(prompts: TestPrompt[], models: ModelConfiguration[], runsPerPrompt: number): Promise<GeneratedResult[]>`**

Phase 1 생성을 실행하는 공개 메서드이다.

1. `totalJobs = prompts.length * models.length * runsPerPrompt`를 계산한다.
2. `setInterval(1000)`으로 진행률 모니터를 시작한다. 매 초 `process.stderr`에 `[Phase 1] Progress: <pct>% | ...` 형태로 출력한다.
3. 모델 → 프롬프트 → 실행 번호(1부터 `runsPerPrompt`까지)의 3중 루프를 돌며 각 조합에 대해 `runJob(model, prompt, i)` Promise를 생성한다.
4. 각 Promise 완료 시 `result.error`가 있으면 `failedCount`를, 없으면 `completedCount`를 증가시킨다.
5. `Promise.all(promises)`로 모든 작업 완료를 기다린다.
6. 인터벌을 정리하고 `\n`을 출력한다.
7. `GeneratedResult[]`를 반환한다.

**`private runJob(model: ModelConfiguration, prompt: TestPrompt, runIndex: number, retryCount: number = 0): Promise<GeneratedResult>`**

단일 생성 작업을 실행하는 비공개 메서드이다.

1. `componentGeneratorFlow({ prompt: prompt.promptText, modelConfig: model, schemas, catalogRules })`를 호출한다.
2. 반환된 `output.text`에서 `extractJsonFromMarkdown(text)`를 호출하여 JSON 배열(`components`)을 추출한다.
3. 추출 성공 시 `outputDir`가 있으면 `saveArtifacts`를 호출한다.
4. 추출 오류 시 `error`를 설정하고 `outputDir`가 있으면 `saveError`를 호출한다.
5. `output.text`가 없으면 `error = new Error('No output text returned from model')`을 설정한다.
6. `{ modelName, prompt, runNumber: runIndex, rawText: text, components, latency, error }`를 반환한다.
7. 외부 예외(플로우 오류 등) 발생 시: `retryCount < 1`이면 `runJob(..., retryCount + 1)`로 1회 재시도한다. 재시도에서도 실패하면 `error`가 포함된 결과 객체를 반환한다.

**`private saveArtifacts(model, prompt, runIndex, text, components)`**

성공한 생성 결과를 파일로 저장하는 비공개 메서드이다.

1. `<outputDir>/output-<modelName>/details/` 디렉토리를 재귀적으로 생성한다.
2. `<promptName>.<runIndex>.json` 파일에 `components` 배열을 JSON으로 저장한다.
3. `<promptName>.<runIndex>.sample` 파일에 YAML front matter와 JSONL 본문을 합쳐서 저장한다. YAML 헤더는 `description`, `name`, `prompt` (들여쓰기된 여러 줄) 필드를 포함하며 `---`로 구분된다. 본문은 각 컴포넌트를 한 줄씩 JSON으로 직렬화한 JSONL이다.

**`private saveError(model, prompt, runIndex, text, error)`**

생성 오류 결과를 파일로 저장하는 비공개 메서드이다.

1. `<outputDir>/output-<modelName>/details/` 디렉토리를 재귀적으로 생성한다.
2. `<promptName>.<runIndex>.output.txt` 파일에 원본 텍스트(없으면 `'No output'`)를 저장한다.
3. `<promptName>.<runIndex>.error.json` 파일에 `{ message, stack }` 객체를 JSON으로 저장한다.

## 동작 흐름

`index.ts`의 `main`에서 호출됨 → 모든 모델/프롬프트/실행 조합에 대해 병렬 생성 → Markdown에서 JSON 추출 → 아티팩트/오류 파일 저장 → `GeneratedResult[]` 반환.
