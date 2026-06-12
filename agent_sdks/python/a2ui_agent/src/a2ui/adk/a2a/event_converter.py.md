# agent_sdks/python/a2ui_agent/src/a2ui/adk/a2a/event_converter.py

## 개요

ADK(Agent Development Kit) 이벤트를 A2A 이벤트로 변환하는 과정에서 A2UI 카탈로그 정보를 자동으로 주입하는 이벤트 컨버터 모듈이다. 세션 상태에서 A2UI 카탈로그를 동적으로 읽어 `A2uiPartConverter`와 연결함으로써, 카탈로그가 세션별로 다른 경우에도 텍스트 기반 A2UI 추출 및 검증이 정상 동작하도록 한다. `@experimental` 데코레이터가 적용되어 있다.

## 의존성

### 외부 패키지
- `typing` — `TYPE_CHECKING`, `Optional`
- `google.adk.a2a.converters.part_converter` — `convert_genai_part_to_a2a_part` (기본 파트 컨버터)
- `google.adk.utils.feature_decorator.experimental` — 실험적 기능 데코레이터
- `google.adk.a2a.converters.event_converter.convert_event_to_a2a_events` — 실제 이벤트 변환 함수 (지연 import)

### 저장소 내부 모듈
- [`a2ui.adk.a2a.part_converter`](./part_converter.py.md) — `A2uiPartConverter`

### TYPE_CHECKING 전용
- `a2a.server.events.Event as A2AEvent`
- `google.adk.a2a.converters.part_converter.GenAIPartToA2APartConverter`
- `google.adk.agents.invocation_context.InvocationContext`
- `google.adk.events.event.Event`

## Exports

- `A2uiEventConverter` (클래스, `@experimental`)

## 상세 명세

### `A2uiEventConverter`

`@experimental` 데코레이터가 적용된 이벤트 컨버터 클래스. callable 객체로 설계되어 ADK의 `A2aAgentExecutorConfig`의 `event_converter` 파라미터에 직접 전달할 수 있다.

#### `__init__(self, catalog_key: str = "system:a2ui_catalog", bypass_tool_check: bool = False, fallback_text: Optional[str] = None)`

- `self._catalog_key`: 세션 상태에서 A2UI 카탈로그를 조회할 때 사용하는 키. 기본값 `"system:a2ui_catalog"`.
- `self._bypass_tool_check`: `A2uiPartConverter` 생성 시 전달될 도구 검증 우회 플래그. 기본값 `False`.
- `self._fallback_text`: `A2uiPartConverter` 생성 시 전달될 파싱 실패 시 대체 텍스트. 기본값 `None`.

#### `__call__(self, event: "Event", invocation_context: "InvocationContext", task_id: Optional[str] = None, context_id: Optional[str] = None, part_converter_func: "GenAIPartToA2APartConverter" = part_converter.convert_genai_part_to_a2a_part) -> list["A2AEvent"]`

ADK 이벤트를 A2A 이벤트 목록으로 변환한다.

1. `invocation_context.session.state.get(self._catalog_key)`로 세션 상태에서 카탈로그를 조회한다.
2. 카탈로그가 존재하면, 해당 카탈로그와 저장된 설정(`bypass_tool_check`, `fallback_text`)으로 `A2uiPartConverter`를 생성하고 그 `.convert` 메서드를 `effective_converter`로 사용한다.
3. 카탈로그가 없으면, 인자로 받은 `part_converter_func`(기본: `convert_genai_part_to_a2a_part`)를 `effective_converter`로 사용한다.
4. `convert_event_to_a2a_events(event, invocation_context, task_id, context_id, effective_converter)`를 호출해 결과를 반환한다. `convert_event_to_a2a_events`는 이 메서드 내에서 지연 import된다.

## 동작 흐름

ADK 에이전트 실행기 설정 시 `A2uiEventConverter()` 인스턴스를 `event_converter`로 등록한다. 각 이벤트가 발생할 때 `__call__`이 호출되어, 세션에 저장된 A2UI 카탈로그를 가져와 카탈로그 인식 파트 컨버터를 동적으로 구성하고, ADK의 표준 이벤트 변환 파이프라인에 주입한다.
