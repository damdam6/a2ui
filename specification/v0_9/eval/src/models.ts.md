# specification/v0_9/eval/src/models.ts

## 개요

평가 파이프라인에서 테스트할 LLM 모델 목록과 각 모델의 메타데이터(이름, 설정, 레이트 리밋)를 정의하는 파일이다. `ModelConfiguration` 인터페이스를 통해 모델 설정의 형태를 명시하고, `modelsToTest` 배열에 OpenAI, Google Gemini, Anthropic Claude 계열의 구체적 모델 항목들을 담는다.

## 의존성

### 외부 패키지
- `@genkit-ai/google-genai` — `googleAI` (Google Gemini 모델 팩토리)
- `@genkit-ai/compat-oai/openai` — `openAI` (OpenAI 호환 모델 팩토리)
- `genkitx-anthropic` — `claude35Haiku`, `claude4Sonnet` (Anthropic 모델 상수)

### 저장소 내부 모듈
없음

## Exports

| 이름 | 종류 | 설명 |
|------|------|------|
| `ModelConfiguration` | 인터페이스 | 단일 모델 설정의 형태 |
| `modelsToTest` | 상수 배열 (`ModelConfiguration[]`) | 평가에 사용할 모든 모델 항목 |

## 상세 명세

### `ModelConfiguration` 인터페이스 (export)

| 필드 | 타입 | 필수 여부 | 설명 |
|------|------|-----------|------|
| `model` | `any` | 필수 | Genkit 모델 레퍼런스 객체 |
| `name` | `string` | 필수 | 모델 식별용 이름 문자열 (레이트 리밋 상태 키로도 사용) |
| `config` | `any` | 선택 | 모델 호출 시 전달할 설정 객체 (예: `reasoning_effort`, `thinkingConfig`) |
| `requestsPerMinute` | `number` | 선택 | 분당 최대 요청 수 |
| `tokensPerMinute` | `number` | 선택 | 분당 최대 토큰 수 |

### `modelsToTest: ModelConfiguration[]` (export)

10개의 모델 항목을 담은 배열. 각 항목의 세부 값은 다음과 같다.

| `name` | `model` 생성 방법 | `config` 주요 값 | `requestsPerMinute` | `tokensPerMinute` |
|--------|-------------------|------------------|---------------------|-------------------|
| `'gpt-5.1'` | `openAI.model('gpt-5.1')` | `{reasoning_effort: 'minimal'}` | 500 | 30000 |
| `'gpt-5-mini'` | `openAI.model('gpt-5-mini')` | `{reasoning_effort: 'minimal'}` | 500 | 500000 |
| `'gpt-5-nano'` | `openAI.model('gpt-5-nano')` | `{}` | 500 | 200000 |
| `'gemini-2.5-pro'` | `googleAI.model('gemini-2.5-pro')` | `{thinkingConfig: {thinkingBudget: 1000}}` | 150 | 2000000 |
| `'gemini-3-flash'` | `googleAI.model('gemini-3-flash-preview')` | `{thinkingConfig: {thinkingBudget: 0}}` | 1000 | 1000000 |
| `'gemini-3.1-pro'` | `googleAI.model('gemini-3.1-pro-preview')` | `{thinkingConfig: {thinkingBudget: 1000}}` | 25 | 1000000 |
| `'gemini-2.5-flash'` | `googleAI.model('gemini-2.5-flash')` | `{thinkingConfig: {thinkingBudget: 0}}` | 1000 | 1000000 |
| `'gemini-2.5-flash-lite'` | `googleAI.model('gemini-2.5-flash-lite')` | `{thinkingConfig: {thinkingBudget: 0}}` | 4000 | 4000000 |
| `'claude-4-sonnet'` | `claude4Sonnet` (상수 직접 사용) | `{}` | 50 | 30000 |
| `'claude-35-haiku'` | `claude35Haiku` (상수 직접 사용) | `{}` | 50 | 50000 |

## 동작 흐름

모듈 로드 시 모든 모델 팩토리 호출이 즉시 실행되어 `modelsToTest` 배열이 구성된다. 이 배열은 평가 파이프라인의 다른 모듈(예: `rateLimiter`, 메인 평가 루프)에서 import하여 반복하며 각 모델에 대해 프롬프트를 실행한다.
