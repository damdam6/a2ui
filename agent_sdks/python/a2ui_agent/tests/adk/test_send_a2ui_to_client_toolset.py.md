# agent_sdks/python/a2ui_agent/tests/adk/test_send_a2ui_to_client_toolset.py

## 개요

`SendA2uiToClientToolset` 클래스와 그 내부 클래스 `_SendA2uiJsonToClientTool`의 동작을 검증하는 단위 테스트 모음이다. toolset 초기화 시 bool·동기 callable·async callable 세 가지 형태의 인자를 처리하는 방식, 도구 활성화/비활성화 로직, 도구 선언(declaration) 구조, LLM 요청 처리(`process_llm_request`), 그리고 `run_async`의 성공/실패 경로(누락 인자, 잘못된 JSON, 스키마 검증 실패)를 전반적으로 커버한다.

## 의존성

### 외부 패키지
- `json` (표준 라이브러리)
- `unittest.mock` — `MagicMock`
- `pytest`, `pytest-asyncio` (`@pytest.mark.asyncio`)
- `google.adk.agents.readonly_context.ReadonlyContext`
- `google.adk.tools.tool_context.ToolContext`
- `google.genai.types` (`genai_types` alias)

### 저장소 내부 모듈
- [`agent_sdks/python/a2ui_agent/src/a2ui/adk/send_a2ui_to_client_toolset.py`](../../src/a2ui/adk/send_a2ui_to_client_toolset.py.md) — `SendA2uiToClientToolset`
- [`agent_sdks/python/a2ui_agent/src/a2ui/schema/catalog.py`](../../src/a2ui/schema/catalog.py.md) — `A2uiCatalog`

## Exports

테스트 함수만 포함하며 외부로 내보내는 심볼은 없다.

## 테스트 케이스 상세 명세

### 영역 1: `SendA2uiToClientToolset` 초기화 및 resolve 메서드

#### `test_toolset_init_bool` (async)
- **검증 동작**: `a2ui_enabled=True`, `a2ui_catalog=catalog_mock`, `a2ui_examples="examples"`로 생성한 toolset에서 `_resolve_a2ui_enabled(ctx)`가 `True`를 반환하는지, 내부 `_ui_tools[0]._resolve_a2ui_catalog(ctx)`가 `catalog_mock`을 반환하는지 확인한다.
- **픽스처/모킹**: `A2uiCatalog` mock, `ReadonlyContext` mock.

#### `test_toolset_init_callable` (async)
- **검증 동작**: `a2ui_enabled=enabled_mock`, `a2ui_catalog=catalog_mock`, `a2ui_examples=examples_mock`으로 생성했을 때(enabled, examples는 동기 callable, catalog는 일반 객체) `_resolve_a2ui_enabled`와 `_resolve_a2ui_examples`가 각각 callable을 1회 호출하여 올바른 값을 반환하고, `catalog_mock`은 callable이 아니므로 직접 호출되지 않음을 확인한다.
- **픽스처/모킹**: `enabled_mock = MagicMock(return_value=True)`, `catalog_mock = MagicMock(spec=A2uiCatalog)`, `examples_mock = MagicMock(return_value="examples")`.

#### `test_toolset_init_async_callable` (async)
- **검증 동작**: `a2ui_enabled`, `a2ui_catalog`, `a2ui_examples`를 모두 로컬 정의 async 함수로 전달해도 `await` 후 올바른 값이 반환되는지 확인한다.
- **픽스처/모킹**: `async_enabled`, `async_catalog`, `async_examples` — 로컬 정의 async 함수.

### 영역 2: `get_tools` 동작

#### `test_toolset_get_tools_enabled` (async)
- **검증 동작**: `a2ui_enabled=True`인 toolset의 `get_tools(ctx)` 결과가 길이 1이고 해당 항목이 `_SendA2uiJsonToClientTool` 인스턴스인지 확인한다.

#### `test_toolset_get_tools_disabled` (async)
- **검증 동작**: `a2ui_enabled=False`인 toolset의 `get_tools(ctx)` 결과가 빈 리스트(길이 0)인지 확인한다.

### 영역 3: `_SendA2uiJsonToClientTool` 세부 테스트

#### `test_send_tool_init`
- **검증 동작**: `_SendA2uiJsonToClientTool(catalog_mock, "examples")` 생성 후 `tool.name`이 `TOOL_NAME` 상수와 같은지, `_a2ui_catalog` 및 `_a2ui_examples` 필드가 올바르게 저장되는지 확인한다.

#### `test_send_tool_get_declaration`
- **검증 동작**: `_get_declaration()` 반환값이 `None`이 아니며, `declaration.name`이 `TOOL_NAME`과 같고, `parameters.properties`와 `parameters.required` 모두에 `A2UI_JSON_ARG_NAME`이 포함되어 있는지 확인한다.

#### `test_send_tool_resolve_catalog` (async)
- **검증 동작**: `_resolve_a2ui_catalog(ctx)`가 생성자에 전달된 `catalog_mock`을 반환하는지 확인한다.

#### `test_send_tool_resolve_examples` (async)
- **검증 동작**: `_resolve_a2ui_examples(ctx)`가 생성자에 전달된 `"examples"` 문자열을 반환하는지 확인한다.

#### `test_send_tool_process_llm_request` (async)
- **검증 동작**: `process_llm_request(tool_context, llm_request)` 호출 시 `llm_request.append_instructions`가 정확히 1회 호출되고, 전달된 인자 문자열에 `"rendered_catalog"`와 `"examples"`가 모두 포함되어 있는지 확인한다.
- **픽스처/모킹**: `ToolContext` mock(`state={}`), `llm_request_mock(append_instructions = MagicMock())`, `catalog_mock.render_as_llm_instructions.return_value = "rendered_catalog"`.

#### `test_send_tool_run_async_valid` (async)
- **검증 동작**: `A2UI_JSON_ARG_NAME` 키에 `json.dumps([{"type": "Text", "text": "Hello"}])`를 담아 `run_async` 호출 시 반환 딕셔너리에 `VALIDATED_A2UI_JSON_KEY`가 있고 값이 파싱된 리스트와 같은지, `tool_context.actions.skip_summarization`이 `True`로 설정되는지, `catalog_mock.validator.validate`가 1회 호출되는지 확인한다.
- **픽스처/모킹**: `catalog_mock.validator.validate.return_value = None`.

#### `test_send_tool_run_async_valid_list` (async)
- **검증 동작**: `test_send_tool_run_async_valid`와 동일한 입력·검증을 수행하는 중복 확인 케이스이다.

#### `test_send_tool_run_async_missing_arg` (async)
- **검증 동작**: args 딕셔너리가 빈 `{}`일 때 반환값에 `"error"` 키가 있고 그 값에 `A2UI_JSON_ARG_NAME`이 포함되어 있는지 확인한다.

#### `test_send_tool_run_async_invalid_json` (async)
- **검증 동작**: `A2UI_JSON_ARG_NAME`에 `"{invalid"` (파싱 불가 JSON)을 전달하면 `"error"` 키가 존재하고 에러 메시지에 `"Failed to call A2UI tool"`과 `"Expecting property name enclosed in double quotes"` 문자열이 모두 포함되는지 확인한다.

#### `test_send_tool_run_async_schema_validation_fail` (async)
- **검증 동작**: `catalog_mock.validator.validate.side_effect = Exception("'text' is a required property")`로 설정 후, `[{"type": "Text"}]`(`text` 필드 누락)를 입력하면 `"error"` 키가 존재하고 에러 메시지에 `"Failed to call A2UI tool"`과 `"'text' is a required property"`가 포함되는지 확인한다.

## 동작 흐름

파일은 두 영역(`# region`)으로 구분된다. 첫 번째 영역(`SendA2uiToClientToolset Tests`)은 toolset 레벨의 초기화 및 tool 목록 반환을 다루고, 두 번째 영역(`SendA2uiJsonToClientTool Tests`)은 개별 도구의 선언 구조, resolve 로직, LLM 요청 처리, 실행 결과를 세부적으로 검증한다. async 테스트는 `@pytest.mark.asyncio`를 사용하며, 모든 외부 의존성은 `MagicMock`으로 대체된다.
