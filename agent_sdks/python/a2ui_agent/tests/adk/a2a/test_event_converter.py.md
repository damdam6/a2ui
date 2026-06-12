# agent_sdks/python/a2ui_agent/tests/adk/a2a/test_event_converter.py

## 개요

`A2uiEventConverter` 클래스의 동작을 검증하는 pytest 테스트 모듈이다. 세션 상태에 카탈로그가 있을 때의 파트 컨버터 주입 경로, 카탈로그 없을 때의 폴백 경로, `fallback_text` 전파 동작을 각각 독립된 테스트 케이스로 검증한다. 외부 ADK 라이브러리의 `convert_event_to_a2a_events` 함수를 `unittest.mock.patch`로 모킹해 실제 이벤트 변환 없이 내부 호출 인수만 검사한다.

## 의존성

### 외부 패키지
- `unittest.mock` — `MagicMock`, `patch`
- `pytest`
- `google.adk.a2a.converters.event_converter.convert_event_to_a2a_events` (모킹 대상)
- `google.adk.a2a.converters.part_converter.convert_genai_part_to_a2a_part` (폴백 검증 대상)

### 저장소 내부 모듈
- [`a2ui.adk.a2a.event_converter`](../../../src/a2ui/adk/a2a/event_converter.py.md) — `A2uiEventConverter`
- [`a2ui.adk.a2a.part_converter`](../../../src/a2ui/adk/a2a/part_converter.py.md) — `A2uiPartConverter`
- [`a2ui.schema.catalog`](../../../src/a2ui/schema/catalog.py.md) — `A2uiCatalog`

## 테스트 케이스

### `test_event_converter_injects_catalog()`

**검증하는 동작:** 세션 상태(`session.state`)에 `"system:a2ui_catalog"` 키로 `A2uiCatalog` 스펙의 MagicMock이 존재할 때, `A2uiEventConverter`가 `convert_event_to_a2a_events`의 다섯 번째 위치 인수로 `A2uiPartConverter` 인스턴스의 바인딩 메서드를 전달하는지 확인한다. 해당 메서드 이름이 `"convert"`이고, `__self__`가 `A2uiPartConverter`의 인스턴스이며, `__self__._catalog`이 주입한 카탈로그 mock과 동일한 객체임을 단언한다.

**픽스처/모킹:**
- `catalog_mock = MagicMock(spec=A2uiCatalog)` — 카탈로그 대역
- `event_mock = MagicMock()` — 이벤트 대역
- `invocation_context_mock.session.state = {"system:a2ui_catalog": catalog_mock}` — 세션 상태 설정
- `patch("google.adk.a2a.converters.event_converter.convert_event_to_a2a_events")` — 기저 변환 함수 모킹, 반환값 `[]`

---

### `test_event_converter_falls_back_without_catalog()`

**검증하는 동작:** 세션 상태가 빈 dict(`{}`)일 때, 즉 카탈로그가 없을 때, `A2uiEventConverter`가 `convert_event_to_a2a_events`의 다섯 번째 위치 인수로 `A2uiPartConverter` 대신 ADK 기본 `convert_genai_part_to_a2a_part` 함수를 전달하는지 확인한다. `effective_part_converter == convert_genai_part_to_a2a_part`로 단언한다.

**픽스처/모킹:**
- `event_mock = MagicMock()`, `invocation_context_mock.session.state = {}` — 카탈로그 없는 컨텍스트
- `patch("google.adk.a2a.converters.event_converter.convert_event_to_a2a_events")` — 기저 변환 함수 모킹
- `from google.adk.a2a.converters.part_converter import convert_genai_part_to_a2a_part` — 테스트 함수 내부에서 임포트해 기댓값으로 사용

---

### `test_event_converter_propagates_fallback_text()`

**검증하는 동작:** `A2uiEventConverter(fallback_text="Custom event fallback text")`처럼 `fallback_text`를 지정해 생성한 경우, 주입된 `A2uiPartConverter` 인스턴스의 `_fallback_text` 속성이 해당 문자열과 일치하는지 확인한다.

**픽스처/모킹:**
- `catalog_mock = MagicMock(spec=A2uiCatalog)`, `invocation_context_mock.session.state = {"system:a2ui_catalog": catalog_mock}` — 카탈로그 있는 컨텍스트
- `custom_fallback = "Custom event fallback text"` — 검증할 fallback 문자열
- `converter = A2uiEventConverter(fallback_text=custom_fallback)` — 생성자 파라미터 전달
- `patch("google.adk.a2a.converters.event_converter.convert_event_to_a2a_events")` — 기저 변환 함수 모킹

## 동작 흐름

모든 테스트는 `A2uiEventConverter` 인스턴스를 직접 호출(`converter(event_mock, invocation_context_mock)`)하고, 모킹된 `convert_event_to_a2a_events`가 수신한 `call_args`를 검사한다. 세 케이스 모두 비동기가 아닌 동기 호출로 실행된다. 카탈로그 존재 여부와 `fallback_text` 파라미터에 따라 다섯 번째 인수(part converter)가 달라지는 분기를 커버한다.
