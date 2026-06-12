# specification/v0_8/eval/src/flows.ts

## 개요

이 파일은 Genkit 인스턴스(`ai`)와 UI 컴포넌트 생성 플로우(`componentGeneratorFlow`)를 정의한다. 환경 변수를 검사하여 사용 가능한 AI 공급자(Google AI, OpenAI, Anthropic)의 플러그인만 동적으로 등록하고, JSON 스키마 기반의 구조화된 출력을 생성하는 단일 Genkit 플로우를 export한다.

## 의존성

### 외부 패키지
- `@genkit-ai/google-genai` — `googleAI` 플러그인 팩토리
- `genkit` — `genkit`, `z` (Zod 스키마 빌더)
- `@genkit-ai/compat-oai/openai` — `openAI` 플러그인 팩토리
- `genkitx-anthropic` — `anthropic` 플러그인 팩토리

### 저장소 내부 모듈
없음

## Exports

| 이름 | 종류 | 설명 |
|------|------|------|
| `ai` | `Genkit` 인스턴스 상수 | 동적으로 구성된 Genkit 인스턴스 |
| `componentGeneratorFlow` | Genkit 플로우 | JSON 스키마 기반 UI 컴포넌트 생성 플로우 |

## 상세 명세

### 플러그인 동적 등록

모듈 최상위에서 `plugins` 빈 배열을 선언하고 세 조건을 순서대로 평가한다.

- `process.env.GEMINI_API_KEY`가 truthy이면 `googleAI({apiKey: process.env.GEMINI_API_KEY!, experimental_debugTraces: true})`를 배열에 추가하고 초기화 로그를 출력한다.
- `process.env.OPENAI_API_KEY`가 truthy이면 `openAI()`를 배열에 추가하고 초기화 로그를 출력한다.
- `process.env.ANTHROPIC_API_KEY`가 truthy이면 `anthropic({apiKey: process.env.ANTHROPIC_API_KEY!})`를 배열에 추가하고 초기화 로그를 출력한다.

### `ai` (상수, Genkit 인스턴스)

`genkit({plugins})`를 호출하여 구성된 Genkit 인스턴스. 위에서 조건부로 구성된 `plugins` 배열을 사용한다.

### `componentGeneratorFlow` (Genkit 플로우)

`ai.defineFlow()`로 정의된다.

**입력 스키마** (`z.object`):
- `prompt: z.string()` — LLM에 전달할 텍스트 프롬프트.
- `model: z.any()` — 사용할 모델 참조 객체.
- `config: z.any().optional()` — 선택적 모델 생성 설정.
- `schema: z.any()` — 출력을 강제할 JSON Schema 객체.

**출력 스키마**: `z.any()`

**실행 로직**:

1. `ai.generate()`를 호출하며 `prompt`, `model`, `config`를 전달하고 `output: {contentType: 'application/json', jsonSchema: schema}`를 설정하여 JSON Schema 기반의 구조화된 출력을 요청한다.
2. `output`이 `null` 또는 falsy이면 `new Error('Failed to generate component')`를 throw한다.
3. 정상 출력이면 `output`을 그대로 반환한다.

## 동작 흐름

모듈 로드 시 환경 변수를 읽어 사용 가능한 플러그인만 등록한 `ai` 인스턴스가 초기화된다. `componentGeneratorFlow`는 `index.ts`에서 각 모델·프롬프트 조합에 대해 호출되며, 주어진 JSON 스키마를 준수하는 JSON 객체를 생성하여 반환한다.
