# specification/v1_0/eval/src/generator.ts

## 개요

여러 모델과 프롬프트 조합에 대해 UI 컴포넌트 생성을 병렬로 실행하는 `Generator` 클래스를 제공한다. `componentGeneratorFlow`를 호출하여 각 (모델, 프롬프트, 실행 번호) 조합에 대한 생성 작업을 수행하고, 결과 텍스트에서 JSON을 추출하며, 성공/실패 아티팩트를 파일 시스템에 저장한다. 단일 재시도 로직을 내장한다.

## 의존성

### 외부 패키지
- `fs` (Node.js 내장)
- `path` (Node.js 내장)

### 저장소 내부 모듈
- [`./generation_flow`](./generation_flow.ts.md) — `componentGeneratorFlow`
- [`./models`](./models.ts.md) — `ModelConfiguration` 타입
- [`./prompts`](./prompts.ts.md) — `TestPrompt` 타입
- [`./types`](./types.ts.md) — `GeneratedResult`, `ProtocolSchemas`
- [`./utils`](./utils.ts.md) — `extractJsonFromMarkdown`
- [`./rateLimiter`](./rateLimiter.ts.md)
- [`./logger`](./logger.ts.md)

## Exports

| 이름 | 종류 | 설명 |
|------|------|------|
| `Generator` | 클래스 | 병렬 UI 생성 실행과 결과 저장을 담당하는 클래스 |

## 상세 명세

### `Generator` 클래스

#### 생성자

```
constructor(
  private schemas: ProtocolSchemas,
  private outputDir?: string,
  private catalogRules?: string
)
```

- `schemas`: `componentGeneratorFlow`에 전달할 프로토콜 스키마 맵
- `outputDir`: 선택적. 아티팩트를 저장할 기본 출력 디렉토리
- `catalogRules`: 선택적. 카탈로그별 추가 지침 문자열

#### `run(prompts, models, runsPerPrompt): Promise<GeneratedResult[]>` (public)

- **매개변수:**
  - `prompts: TestPrompt[]`
  - `models: ModelConfiguration[]`
  - `runsPerPrompt: number`
- **반환:** `Promise<GeneratedResult[]>`

**동작 단계:**

1. `totalJobs = prompts.length * models.length * runsPerPrompt` 계산.

2. 1초 간격의 `setInterval`로 `[Phase 1] Progress: <pct>% | Completed: <n> | In Progress: <n> | Queued: <n> | Failed: <n>` 형식의 진행 상황을 `process.stderr`에 출력. `rateLimiter.waitingCount`로 대기 수를 읽는다.

3. 세 겹의 for 루프로 모든 `(model, prompt, runIndex 1..runsPerPrompt)` 조합에 대해 `this.runJob(model, prompt, i)` Promise를 생성하고 배열에 push한다. 각 Promise 완료 콜백에서 `result.error` 유무에 따라 `failedCount` 또는 `completedCount`를 증가시키고 `results`에 추가한다.

4. `Promise.all(promises)`로 모든 작업을 병렬 실행.

5. 완료 후 인터벌 클리어, 개행 출력, 완료 로그 기록 후 `results` 반환.

#### `runJob(model, prompt, runIndex, retryCount = 0): Promise<GeneratedResult>` (private)

- **매개변수:**
  - `model: ModelConfiguration`
  - `prompt: TestPrompt`
  - `runIndex: number`
  - `retryCount: number` — 기본값 `0`

**동작 단계:**

1. `Date.now()`로 시작 시각 기록.

2. `componentGeneratorFlow({ prompt: prompt.promptText, modelConfig: model, schemas: this.schemas, catalogRules: this.catalogRules })`를 await 호출.

3. 응답에서 `text`와 `latency`를 추출한다. `latency`가 없으면 `0` 사용.

4. `text`가 있으면 `extractJsonFromMarkdown(text)`로 JSON 컴포넌트 배열을 파싱한다. 성공하면 `outputDir`가 있을 때 `saveArtifacts`를 호출. 파싱 실패(예외)하면 `error`를 기록하고 `outputDir`가 있을 때 `saveError`를 호출.

5. `text`가 없으면 `error = new Error('No output text returned from model')`을 설정.

6. `{ modelName: model.name, prompt, runNumber: runIndex, rawText: text, components, latency, error }` 형태의 `GeneratedResult`를 반환.

7. 최상위 예외(flow 호출 자체 실패) 시: `retryCount < 1`이면 `runJob(model, prompt, runIndex, retryCount + 1)`을 재귀 호출(1회 재시도). 재시도 소진 시 `{ modelName, prompt, runNumber, latency: Date.now() - startTime, error }` 반환(components 없음).

#### `saveArtifacts(model, prompt, runIndex, text, components)` (private)

1. `<outputDir>/output-<modelName>/details/` 디렉토리를 `fs.mkdirSync({ recursive: true })`로 생성.
2. `<promptName>.<runIndex>.json` 파일에 `JSON.stringify(components, null, 2)` 저장.
3. `<promptName>.<runIndex>.sample` 파일을 저장:
   - YAML front matter: `description`, `name`, `prompt` (각 줄에 2칸 들여쓰기) 포함
   - 본문: `components` 배열의 각 항목을 `JSON.stringify(comp) + '\n'`으로 이어붙인 JSONL 형식

#### `saveError(model, prompt, runIndex, text, error)` (private)

1. `<outputDir>/output-<modelName>/details/` 디렉토리를 `fs.mkdirSync({ recursive: true })`로 생성.
2. `<promptName>.<runIndex>.output.txt`에 `text || 'No output'` 저장.
3. `<promptName>.<runIndex>.error.json`에 `{ message, stack }` 저장.

## 동작 흐름

`index.ts`의 `main`에서 Phase 1로 호출된다. 모든 (모델 × 프롬프트 × 실행 번호) 조합의 Promise를 한 번에 생성하여 `Promise.all`로 병렬 실행한다. 개별 실패는 재시도 후에도 실패 시 error 필드가 있는 결과로 기록되며, 전체 흐름은 중단되지 않는다. 결과는 `Validator`의 Phase 2 입력으로 전달된다.
