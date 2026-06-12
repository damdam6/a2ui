# agent_sdks/python/a2ui_agent/src/a2ui/template/manager.py

## 개요

템플릿 기반 추론 전략을 구현하기 위한 `A2uiTemplateManager` 클래스를 정의한다. `InferenceStrategy` 인터페이스를 상속하며, LLM 시스템 프롬프트를 생성하는 `generate_system_prompt` 메서드의 시그니처를 갖추고 있으나 현재는 미구현 상태(`NotImplementedError`)다. 향후 템플릿 기반 시스템 프롬프트 생성 로직이 채워질 예정이다.

## 의존성

### 외부 패키지
- `typing` — `Optional`, `Any`

### 저장소 내부 모듈
- [`../inference_strategy`](../inference_strategy.py.md) — `InferenceStrategy` 추상 클래스

## Exports

- `A2uiTemplateManager` (클래스)

## 상세 명세

### `class A2uiTemplateManager(InferenceStrategy)`

`InferenceStrategy`를 상속하는 클래스. 별도의 `__init__`을 정의하지 않으므로 부모 클래스의 생성자를 그대로 사용한다.

#### `generate_system_prompt(self, role_description, workflow_description="", ui_description="", client_ui_capabilities=None, allowed_components=None, allowed_messages=None, include_schema=False, include_examples=False, validate_examples=False) -> str`

**매개변수:**
- `role_description: str` — 에이전트의 역할 설명 (필수)
- `workflow_description: str` — 워크플로우 설명 (기본값 `""`)
- `ui_description: str` — UI 설명 (기본값 `""`)
- `client_ui_capabilities: Optional[dict[str, Any]]` — 클라이언트 UI 기능 명세 (기본값 `None`)
- `allowed_components: Optional[list[str]]` — 허용된 컴포넌트 목록 (기본값 `None`)
- `allowed_messages: Optional[list[str]]` — 허용된 메시지 타입 목록 (기본값 `None`)
- `include_schema: bool` — 스키마 포함 여부 (기본값 `False`)
- `include_examples: bool` — 예시 포함 여부 (기본값 `False`)
- `validate_examples: bool` — 예시 검증 여부 (기본값 `False`)

**동작:** 현재는 `NotImplementedError("This method is not yet implemented.")`를 발생시킨다. 미래에는 주어진 파라미터를 조합해 LLM용 시스템 프롬프트 문자열을 생성해 반환할 예정이다.

## 동작 흐름

클래스는 인스턴스화 가능하지만, `generate_system_prompt`를 호출하면 항상 `NotImplementedError`가 발생한다. 구현이 완료되면 `InferenceStrategy` 인터페이스 계약에 따라 시스템 프롬프트 문자열을 반환해야 한다.
