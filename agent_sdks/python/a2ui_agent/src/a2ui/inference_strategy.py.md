# agent_sdks/python/a2ui_agent/src/a2ui/inference_strategy.py

## 개요

A2UI 에이전트의 LLM 시스템 프롬프트 생성 전략을 정의하는 추상 기반 클래스 모듈이다. 구체적인 구현체가 역할 설명, 워크플로, UI 설명, 카탈로그 컴포넌트 필터 등을 조합해 시스템 프롬프트를 생성하는 방식을 표준화한다.

## 의존성

### 외부 패키지
- `abc` — `ABC`, `abstractmethod`
- `typing` — `Optional`, `Any`

## Exports

- `InferenceStrategy` (추상 클래스)

## 상세 명세

### `InferenceStrategy(ABC)`

추상 기반 클래스. 단일 추상 메서드 `generate_system_prompt`를 정의한다.

#### `@abstractmethod generate_system_prompt(self, role_description: str, workflow_description: str = "", ui_description: str = "", client_ui_capabilities: Optional[dict[str, Any]] = None, allowed_components: Optional[list[str]] = None, allowed_messages: Optional[list[str]] = None, include_schema: bool = False, include_examples: bool = False, validate_examples: bool = False) -> str`

모든 LLM 요청에 사용할 시스템 프롬프트 문자열을 생성하는 추상 메서드.

매개변수:
- `role_description: str` — 에이전트의 역할 설명 (필수).
- `workflow_description: str = ""` — 워크플로 설명.
- `ui_description: str = ""` — UI 설명.
- `client_ui_capabilities: Optional[dict[str, Any]] = None` — 클라이언트가 보고한 UI 능력 정보(스키마 가지치기에 활용).
- `allowed_components: Optional[list[str]] = None` — 허용된 카탈로그 컴포넌트 목록.
- `allowed_messages: Optional[list[str]] = None` — 허용된 메시지 목록.
- `include_schema: bool = False` — 스키마 포함 여부.
- `include_examples: bool = False` — 예제 포함 여부.
- `validate_examples: bool = False` — 예제 검증 여부.

반환값: 생성된 시스템 프롬프트 문자열. 본체는 `pass`로만 구성되어 있으며, 하위 클래스에서 반드시 오버라이드해야 한다.

## 동작 흐름

`InferenceStrategy`를 상속한 구체 클래스가 `generate_system_prompt`를 구현한다. 구현체는 전달받은 역할·워크플로·UI 설명과 카탈로그 정보를 조합해 LLM에 전달할 완성된 시스템 프롬프트 문자열을 반환한다. 호출자는 구체 구현체의 종류에 관계없이 이 인터페이스를 통해 프롬프트를 요청할 수 있다.
