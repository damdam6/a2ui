# specification/v1_0/eval/src/ai.ts

## 개요

환경 변수에 따라 AI 제공자 플러그인을 동적으로 등록하고 공유 Genkit 인스턴스(`ai`)를 생성하는 파일이다. `GEMINI_API_KEY`, `OPENAI_API_KEY`, `ANTHROPIC_API_KEY` 세 가지 환경 변수 중 존재하는 것만 해당 플러그인을 활성화하며, 등록된 플러그인 목록을 사용해 단일 `genkit` 인스턴스를 초기화한다. 이 인스턴스는 모든 flow 파일에서 공유된다.

## 의존성

### 외부 패키지
- `@genkit-ai/google-genai` — `googleAI` 플러그인 (Gemini 모델 연동)
- `genkit` — `genkit` 팩토리 함수
- `@genkit-ai/compat-oai/openai` — `openAI` 플러그인 (OpenAI 호환 인터페이스)
- `genkitx-anthropic` — `anthropic` 플러그인 (Claude 모델 연동)

### 저장소 내부 모듈
- [`./logger`](./logger.ts.md)

## Exports

| 이름 | 종류 | 설명 |
|------|------|------|
| `ai` | `Genkit` 인스턴스 (const) | 활성화된 플러그인으로 구성된 공유 Genkit 인스턴스 |

## 상세 명세

### 플러그인 등록 로직 (모듈 최상위)

빈 배열 `plugins`를 선언한 뒤 세 가지 환경 변수를 순서대로 검사한다.

1. `process.env.GEMINI_API_KEY`가 존재하면 `googleAI({ apiKey: ..., experimental_debugTraces: true })`를 플러그인 배열에 추가하고 `logger.info`로 초기화 메시지를 출력한다.
2. `process.env.OPENAI_API_KEY`가 존재하면 `openAI()`를 추가하고 로그를 출력한다.
3. `process.env.ANTHROPIC_API_KEY`가 존재하면 `anthropic({ apiKey: process.env.ANTHROPIC_API_KEY! })`를 추가하고 로그를 출력한다.

각 조건은 독립적으로 평가되어 여러 제공자를 동시에 활성화할 수 있다.

### `ai` (const, export)

`genkit({ plugins })` 호출 결과로 생성된 Genkit 인스턴스. `plugins` 배열에는 위 조건부 로직에서 추가된 플러그인만 포함된다. 환경 변수가 하나도 없으면 플러그인 없이 초기화된다.

## 동작 흐름

모듈 로드 시 환경 변수를 검사하여 플러그인 목록을 동적으로 구성한 뒤 `genkit()`로 인스턴스를 생성한다. 이후 `analysis_flow.ts`, `evaluation_flow.ts`, `generation_flow.ts` 등에서 이 `ai` 인스턴스를 import하여 `ai.defineFlow(...)` 및 `ai.generate(...)` 호출에 사용한다.
