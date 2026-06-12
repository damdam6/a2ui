# specification/v0_9/eval/src/ai.ts

## 개요

환경 변수에 따라 조건적으로 AI 플러그인을 등록하고 Genkit 인스턴스(`ai`)를 초기화하는 모듈이다. `GEMINI_API_KEY`, `OPENAI_API_KEY`, `ANTHROPIC_API_KEY` 세 가지 환경 변수를 검사하여, 존재하는 키에 해당하는 플러그인만 활성화한다. 이 `ai` 인스턴스는 프로젝트 전체에서 모델 호출 및 플로우 정의의 공통 진입점으로 사용된다.

## 의존성

### 외부 패키지

- `@genkit-ai/google-genai` — `googleAI` 플러그인 팩토리
- `genkit` — `genkit` 인스턴스 생성 함수
- `@genkit-ai/compat-oai/openai` — `openAI` 호환 플러그인 팩토리
- `genkitx-anthropic` — `anthropic` 플러그인 팩토리

### 저장소 내부 모듈

- [`./logger`](./logger.ts.md) — `logger` 인스턴스

## Exports

- `ai` — 초기화된 `genkit` 인스턴스 (상수)

## 상세 명세

### `ai` (상수, `Genkit` 인스턴스)

파일 최상위 수준에서 실행되는 초기화 블록에 의해 생성된다. 과정은 다음과 같다.

1. 빈 배열 `plugins`를 선언한다.
2. `process.env.GEMINI_API_KEY`가 존재하면 `googleAI({ apiKey: ..., experimental_debugTraces: true })`를 `plugins`에 추가하고, 로그에 `'Initializing Google AI plugin...'`을 출력한다.
3. `process.env.OPENAI_API_KEY`가 존재하면 `openAI()`를 `plugins`에 추가하고, 로그에 `'Initializing OpenAI plugin...'`을 출력한다.
4. `process.env.ANTHROPIC_API_KEY`가 존재하면 `anthropic({ apiKey: ... })`를 `plugins`에 추가하고, 로그에 `'Initializing Anthropic plugin...'`을 출력한다.
5. 위에서 구성한 `plugins` 배열을 사용하여 `genkit({ plugins })` 를 호출하고, 결과를 `ai`로 export한다.

어떤 환경 변수도 설정되지 않은 경우, `plugins`는 빈 배열이며 Genkit 인스턴스는 플러그인 없이 생성된다.

## 동작 흐름

모듈 로드 시점에 환경 변수를 검사하여 플러그인 목록을 동적으로 구성한 뒤, 단일 `ai` 인스턴스를 생성하여 export한다. 이후 모든 플로우 파일은 이 `ai` 인스턴스를 import하여 `ai.defineFlow(...)` 및 `ai.generate(...)` 를 호출한다.
