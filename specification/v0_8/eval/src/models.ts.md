# specification/v0_8/eval/src/models.ts

## 개요

이 파일은 평가 실행에 사용할 AI 모델 목록과 각 모델의 설정을 정의한다. `ModelConfiguration` 인터페이스와 `modelsToTest` 배열을 export하여 `index.ts`가 어떤 모델을 테스트할지 결정하는 데 사용한다. OpenAI, Google AI(Gemini), Anthropic(Claude) 세 공급자의 모델을 포함한다.

## 의존성

### 외부 패키지
- `@genkit-ai/google-genai` — `googleAI` (Google AI 모델 팩토리)
- `@genkit-ai/compat-oai/openai` — `openAI` (OpenAI 호환 모델 팩토리)
- `genkitx-anthropic` — `claude35Haiku`, `claude4Sonnet` (Anthropic 모델 참조)

### 저장소 내부 모듈
없음

## Exports

| 이름 | 종류 | 설명 |
|------|------|------|
| `ModelConfiguration` | interface | 단일 모델 설정을 나타내는 타입 |
| `modelsToTest` | `ModelConfiguration[]` 상수 | 평가 대상 모델 목록 |

## 상세 명세

### `ModelConfiguration` (interface)

세 필드를 가진다.

- `model: any` — Genkit이 수용하는 모델 참조 객체.
- `name: string` — 모델의 사람이 읽을 수 있는 식별 이름. 출력 디렉토리 이름과 CLI `--model` 필터에 사용된다.
- `config?: any` — 선택적 모델별 설정 객체. 생성 파라미터(추론 노력, thinking budget 등)를 담는다.

### `modelsToTest` (상수, `ModelConfiguration[]`)

총 8개의 모델 설정을 포함하는 배열이다. 순서와 각 항목의 값은 다음과 같다.

1. `name: 'gpt-5'` — `openAI.model('gpt-5')`, `config: {reasoning_effort: 'minimal'}`
2. `name: 'gpt-5-mini'` — `openAI.model('gpt-5-mini')`, `config: {reasoning_effort: 'minimal'}`
3. `name: 'gpt-4.1'` — `openAI.model('gpt-4.1')`, `config: {}`
4. `name: 'gemini-2.5-pro-thinking'` — `googleAI.model('gemini-2.5-pro')`, `config: {thinkingConfig: {thinkingBudget: 1000}}`
5. `name: 'gemini-2.5-flash'` — `googleAI.model('gemini-2.5-flash')`, `config: {thinkingConfig: {thinkingBudget: 0}}`
6. `name: 'gemini-2.5-flash-lite'` — `googleAI.model('gemini-2.5-flash-lite')`, `config: {thinkingConfig: {thinkingBudget: 0}}`
7. `name: 'claude-4-sonnet'` — `claude4Sonnet`, `config: {}`
8. `name: 'claude-35-haiku'` — `claude35Haiku`, `config: {}`

Gemini 모델은 thinking budget을 명시적으로 제어한다. `gemini-2.5-pro-thinking`만 `thinkingBudget: 1000`으로 사고 과정을 허용하고, 나머지 Gemini 모델은 `thinkingBudget: 0`으로 비활성화한다.

## 동작 흐름

이 파일은 선언만 수행한다. 모듈이 로드될 때 배열이 초기화되며, `index.ts`에서 이 배열을 참조하여 CLI `--model` 옵션으로 필터링한 뒤 각 모델에 대해 평가 플로우를 실행한다.
