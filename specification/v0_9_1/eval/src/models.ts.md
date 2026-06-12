# specification/v0_9_1/eval/src/models.ts

## 개요

이 파일은 평가 대상 AI 모델 목록과 각 모델의 설정(레이트 리밋 포함)을 정의한다. `ModelConfiguration` 인터페이스와 `modelsToTest` 배열을 내보내며, 평가 파이프라인의 여러 단계에서 이 목록을 순회하여 각 모델에 요청을 보낸다. 현재 OpenAI GPT 계열, Google Gemini 계열, Anthropic Claude 계열 총 10개 모델이 등록되어 있다.

## 의존성

### 외부 패키지
- `@genkit-ai/google-genai` — `googleAI` (Google Gemini 모델 어댑터)
- `@genkit-ai/compat-oai/openai` — `openAI` (OpenAI 호환 모델 어댑터)
- `genkitx-anthropic` — `claude35Haiku`, `claude4Sonnet` (Anthropic Claude 모델 참조)

### 저장소 내부 모듈
없음

## Exports

| 이름 | 종류 |
|---|---|
| `ModelConfiguration` | 인터페이스 |
| `modelsToTest` | `ModelConfiguration[]` 상수 |

## 상세 명세

### `ModelConfiguration` 인터페이스 (export)

평가 파이프라인이 하나의 모델을 다루기 위해 필요한 모든 정보를 담는 구조체.

| 필드 | 타입 | 필수 여부 | 설명 |
|---|---|---|---|
| `model` | `any` | 필수 | Genkit 모델 참조 객체 |
| `name` | `string` | 필수 | 모델의 식별자 문자열 (로그, 파일명, 레이트 리미터 키로 사용) |
| `config` | `any` | 선택 | 모델 호출 시 전달할 추가 설정 객체 |
| `requestsPerMinute` | `number` | 선택 | 분당 최대 요청 수 (RPM 제한) |
| `tokensPerMinute` | `number` | 선택 | 분당 최대 토큰 수 (TPM 제한) |

### `modelsToTest: ModelConfiguration[]` (export)

평가 대상 모델 목록. 배열 순서대로 등록되어 있으며 각 항목의 세부 값은 다음과 같다.

| name | model 생성 방식 | config 주요 내용 | RPM | TPM |
|---|---|---|---|---|
| `'gpt-5.1'` | `openAI.model('gpt-5.1')` | `{reasoning_effort: 'minimal'}` | 500 | 30,000 |
| `'gpt-5-mini'` | `openAI.model('gpt-5-mini')` | `{reasoning_effort: 'minimal'}` | 500 | 500,000 |
| `'gpt-5-nano'` | `openAI.model('gpt-5-nano')` | `{}` | 500 | 200,000 |
| `'gemini-2.5-pro'` | `googleAI.model('gemini-2.5-pro')` | `{thinkingConfig: {thinkingBudget: 1000}}` | 150 | 2,000,000 |
| `'gemini-3-flash'` | `googleAI.model('gemini-3-flash-preview')` | `{thinkingConfig: {thinkingBudget: 0}}` | 1,000 | 1,000,000 |
| `'gemini-3.1-pro'` | `googleAI.model('gemini-3.1-pro-preview')` | `{thinkingConfig: {thinkingBudget: 1000}}` | 25 | 1,000,000 |
| `'gemini-2.5-flash'` | `googleAI.model('gemini-2.5-flash')` | `{thinkingConfig: {thinkingBudget: 0}}` | 1,000 | 1,000,000 |
| `'gemini-2.5-flash-lite'` | `googleAI.model('gemini-2.5-flash-lite')` | `{thinkingConfig: {thinkingBudget: 0}}` | 4,000 | 4,000,000 |
| `'claude-4-sonnet'` | `claude4Sonnet` (직접 참조) | `{}` | 50 | 30,000 |
| `'claude-35-haiku'` | `claude35Haiku` (직접 참조) | `{}` | 50 | 50,000 |

Google Gemini 모델의 경우 `thinkingBudget: 0`이면 추론 단계를 비활성화하고, 양수 값이면 해당 토큰 수만큼 내부 추론 예산을 허용한다.

## 동작 흐름

이 파일은 선언적으로만 동작한다. 모듈 로드 시 모든 모델 인스턴스가 생성되어 `modelsToTest` 배열에 담긴다. 평가 실행 진입점에서 이 배열을 가져와 각 `ModelConfiguration`을 순회하면서 `name`으로 레이트 리미터를 식별하고, `model`과 `config`로 실제 LLM 호출을 수행한다.
