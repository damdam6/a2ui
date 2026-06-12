# agent_sdks/python/a2ui_agent/tests/adk/a2a/test_part_converter.py

## 개요

`A2uiPartConverter` 클래스의 `convert` 메서드가 다양한 `genai_types.Part` 입력을 올바르게 A2A 파트 목록으로 변환하는지 검증하는 단위 테스트 모음이다. FunctionResponse(성공/에러/빈 응답), FunctionCall, 텍스트(A2UI 태그 포함/미포함, 마크다운 래핑), inline_data 등 모든 경로의 변환 동작을 망라하며, 버전 처리(`VERSION_0_8`, `VERSION_0_9_1`) 및 `fallback_text` 동작까지 포함한다.

## 의존성

### 외부 패키지
- `json` (표준 라이브러리)
- `unittest.mock` — `MagicMock`, `patch`
- `pytest`
- `a2a.types` (`a2a_types` alias) — `Part`, `DataPart`
- `google.genai.types` (`genai_types` alias) — `FunctionResponse`, `FunctionCall`, `Part`, `Blob`

### 저장소 내부 모듈
- [`agent_sdks/python/a2ui_agent/src/a2ui/a2a/parts.py`](../../../src/a2ui/a2a/parts.py.md) — `create_a2ui_part`
- [`agent_sdks/python/a2ui_agent/src/a2ui/adk/a2a/part_converter.py`](../../../src/a2ui/adk/a2a/part_converter.py.md) — `A2uiPartConverter`
- [`agent_sdks/python/a2ui_agent/src/a2ui/adk/send_a2ui_to_client_toolset.py`](../../../src/a2ui/adk/send_a2ui_to_client_toolset.py.md) — `SendA2uiToClientToolset`
- [`agent_sdks/python/a2ui_agent/src/a2ui/schema/catalog.py`](../../../src/a2ui/schema/catalog.py.md) — `A2uiCatalog`
- [`agent_sdks/python/a2ui_agent/src/a2ui/schema/constants.py`](../../../src/a2ui/schema/constants.py.md) — `A2UI_CLOSE_TAG`, `A2UI_OPEN_TAG`, `VERSION_0_8`, `VERSION_0_9_1`

## Exports

이 파일은 테스트 함수만 포함하며 외부로 내보내는 심볼은 없다.

## 테스트 케이스 상세 명세

### `test_converter_class_convert_valid_tool_response`
- **검증 동작**: `A2uiPartConverter(catalog_mock)`(버전 기본값 `VERSION_0_8`)가 `_SendA2uiJsonToClientTool.TOOL_NAME`을 이름으로 하고, 응답에 `VALIDATED_A2UI_JSON_KEY: [{"type": "Text", "text": "Hello"}]`를 담은 `FunctionResponse` Part를 변환했을 때, 결과 길이가 1이고 `create_a2ui_part(valid_a2ui, version=VERSION_0_8)` 결과와 동등한지 확인한다.
- **픽스처/모킹**: `A2uiCatalog`를 `MagicMock(spec=A2uiCatalog)`으로 대체.

### `test_converter_class_convert_valid_tool_response_v0_9_1`
- **검증 동작**: 위와 동일하되 `A2uiPartConverter(catalog_mock, version=VERSION_0_9_1)`로 생성 시 결과 파트가 `create_a2ui_part(valid_a2ui, version=VERSION_0_9_1)`과 동등한지 확인한다. 버전 파라미터가 결과 Part 생성에 전파되는지 검증.
- **픽스처/모킹**: `A2uiCatalog` mock.

### `test_converter_class_convert_tool_error_response`
- **검증 동작**: 응답 딕셔너리에 `TOOL_ERROR_KEY: "Some error"`만 있는 `FunctionResponse` Part를 변환하면 결과 리스트가 빈 상태(길이 0)임을 확인한다.
- **픽스처/모킹**: `A2uiCatalog` mock.

### `test_converter_class_convert_tool_response_no_result`
- **검증 동작**: 응답 딕셔너리가 빈 `{}`인 `FunctionResponse` Part를 변환하면 결과 리스트 길이가 0임을 확인한다.
- **픽스처/모킹**: `A2uiCatalog` mock.

### `test_converter_class_convert_function_call_ignores`
- **검증 동작**: `FunctionCall`을 담은 Part(response가 아닌 call 방향)를 변환하면 길이 0의 리스트를 반환하여 컨버터가 function call을 무시함을 확인한다.
- **픽스처/모킹**: `A2uiCatalog` mock.

### `test_converter_class_convert_text_with_a2ui`
- **검증 동작**: `"Here is the UI:\n{A2UI_OPEN_TAG}\n<JSON>\n{A2UI_CLOSE_TAG}"` 형태의 텍스트 Part를 변환하면 2개의 파트가 생성된다. 첫 번째 파트의 `.root.text`가 `"Here is the UI:"`, 두 번째 파트가 `create_a2ui_part(valid_a2ui[0], version=VERSION_0_8)` 결과와 동등한지 확인한다. `catalog_mock.validator.validate`가 `valid_a2ui`를 인수로 정확히 1회 호출됨도 검증한다.
- **픽스처/모킹**: `catalog_mock.validator.validate.return_value = None`.

### `test_converter_class_convert_text_with_a2ui_v0_9_1`
- **검증 동작**: 위와 동일하되 `version=VERSION_0_9_1` 컨버터를 사용하며, DataPart가 `create_a2ui_part(valid_a2ui[0], version=VERSION_0_9_1)`과 동등한지 확인한다.

### `test_converter_class_convert_text_empty_leading`
- **검증 동작**: 텍스트가 `"\n{A2UI_OPEN_TAG}\n<JSON>\n{A2UI_CLOSE_TAG}"`로 태그 앞 내용이 개행문자뿐일 때, 빈/공백 leading text용 TextPart를 생성하지 않고 DataPart 1개만 반환하는지 확인한다.

### `test_converter_class_convert_text_markdown_wrapped`
- **검증 동작**: A2UI 태그 내부 JSON이 `` ```json\n...\n``` `` 마크다운 코드 펜스로 감싸인 경우에도 올바르게 파싱하여 TextPart + DataPart 2개를 반환하는지 확인한다. `catalog_mock.validator.validate`가 파싱된 리스트를 인수로 1회 호출됨을 검증한다.

### `test_converter_class_convert_text_with_invalid_a2ui`
- **검증 동작**: A2UI 태그 내부 내용이 `"invalid_json"`(파싱 불가)일 때, `fallback_text`가 없는 기본 상태에서 결과 리스트가 빈 상태(길이 0)임을 확인한다.

### `test_converter_class_convert_other_part`
- **검증 동작**: `inline_data`(이미지 Blob)를 담은 Part처럼 A2UI 관련이 없는 Part를 변환할 때, `google.adk.a2a.converters.part_converter.convert_genai_part_to_a2a_part`에 위임 호출하고 그 반환값을 1개 그대로 반환하는지 확인한다.
- **픽스처/모킹**: `patch("google.adk.a2a.converters.part_converter.convert_genai_part_to_a2a_part")`로 mock 반환값 `a2a_types.Part(root=a2a_types.DataPart(kind="data", data={}))` 주입. `mock_convert.assert_called_once_with(part)` 검증 포함.

### `test_converter_class_convert_tool_response_with_result_containing_a2ui`
- **검증 동작**: 이름이 `"some_generic_tool"`인 FunctionResponse의 `response["result"]` 값이 A2UI 태그를 포함하는 텍스트일 때, 컨버터가 그 텍스트를 파싱하여 TextPart + DataPart 2개를 반환한다. TextPart의 `.root.text`가 `"Here is the result:"`, DataPart가 `create_a2ui_part(valid_a2ui[0], version=VERSION_0_8)`과 동등한지 확인한다.

### `test_converter_class_convert_text_with_invalid_a2ui_and_custom_fallback`
- **검증 동작**: `fallback_text="Could not build interface."`로 초기화한 컨버터가 유효하지 않은 A2UI JSON 텍스트를 변환했을 때, 결과 1개를 반환하고 해당 파트의 `.root.text`가 fallback 문자열인지 확인한다.

### `test_converter_class_convert_tool_response_with_result_containing_invalid_a2ui_and_default_fallback`
- **검증 동작**: generic FunctionResponse의 `result` 값에 잘못된 A2UI JSON이 포함되어 있고 `fallback_text`를 지정하지 않은 경우 결과가 빈 리스트(길이 0)임을 확인한다.

### `test_converter_class_convert_tool_response_with_result_containing_invalid_a2ui_and_custom_fallback`
- **검증 동작**: 위와 동일하되 `fallback_text="Could not load the custom tool UI."`를 지정한 경우 결과 1개를 반환하고 `.root.text`가 해당 fallback 문자열인지 확인한다.

## 동작 흐름

각 테스트는 독립적으로 동작한다. `A2uiCatalog`를 `MagicMock(spec=A2uiCatalog)`으로 대체한 후 `A2uiPartConverter`를 인스턴스화하고, 다양한 형태의 `genai_types.Part`를 `converter.convert(part)`에 전달한 뒤 반환된 A2A 파트 목록의 길이 및 내용을 pytest assert로 검증한다. 버전 관련 테스트는 `VERSION_0_8`(기본값)와 `VERSION_0_9_1` 두 경로를 각각 별도 함수로 커버하며, fallback 동작은 `fallback_text` 인수 유무에 따른 두 가지 시나리오를 검증한다.
