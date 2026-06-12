# specification/v0_9_1/eval/src/ai.ts

## 개요

환경 변수를 읽어 사용 가능한 AI 제공자 플러그인을 동적으로 등록하고, 단일 `genkit` 인스턴스(`ai`)를 생성하여 내보내는 모듈이다. Google AI, OpenAI, Anthropic 세 가지 제공자를 지원하며, 각 제공자에 해당하는 API 키 환경 변수가 설정된 경우에만 플러그인이 활성화된다. 이 `ai` 인스턴스는 프로젝트 전반에 걸쳐 플로우 정의 및 텍스트 생성에 사용된다.

## 의존성

### 외부 패키지
- `@genkit-ai/google-genai` — `googleAI` 플러그인 팩토리
- `genkit` — `genkit` 인스턴스 생성 함수
- `@genkit-ai/compat-oai/openai` — `openAI` 호환 플러그인 팩토리
- `genkitx-anthropic` — `anthropic` 플러그인 팩토리

### 저장소 내부 모듈
- [`./logger`](./logger.ts.md)

## Exports

- `ai` (상수) — `genkit(...)` 호출로 생성된 Genkit 인스턴스

## 상세 명세

### 플러그인 초기화 로직

모듈 최상위에서 빈 배열 `plugins`를 선언한 뒤 아래 순서로 조건 분기하여 플러그인을 추가한다.

1. `process.env.GEMINI_API_KEY`가 존재하면 `logger.info`로 초기화 시작을 알리고, `googleAI({ apiKey: process.env.GEMINI_API_KEY!, experimental_debugTraces: true })`를 `plugins`에 push한다.
2. `process.env.OPENAI_API_KEY`가 존재하면 `logger.info`로 알리고, 인자 없이 호출한 `openAI()`를 push한다.
3. `process.env.ANTHROPIC_API_KEY`가 존재하면 `logger.info`로 알리고, `anthropic({ apiKey: process.env.ANTHROPIC_API_KEY! })`를 push한다.

어느 환경 변수도 설정되지 않으면 `plugins` 배열은 비어 있으며, `genkit` 인스턴스는 플러그인 없이 생성된다.

### `ai` (상수)

`genkit({ plugins })`를 호출하여 생성된 Genkit 인스턴스. 등록된 플러그인 목록은 위 초기화 로직에서 구성된 `plugins` 배열이다. 이 인스턴스의 `defineFlow`, `generate` 메서드가 프로젝트 전반에서 사용된다.

## 동작 흐름

모듈이 import되는 시점에 환경 변수를 확인하여 플러그인 배열을 구성한 뒤 `genkit` 인스턴스를 생성하고, `ai`라는 이름으로 내보낸다. 이후 다른 모듈은 이 `ai` 인스턴스를 import하여 플로우 정의나 AI 생성 호출에 사용한다.
