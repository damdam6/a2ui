# agent_sdks/python/a2ui_agent/tests/parser/test_streaming_v09.py

## 개요

이 파일은 v0.9 프로토콜 기준으로 `A2uiStreamParser`의 동작을 검증하는 pytest 테스트 모음이다. v0.8 대응 파일(`test_streaming_v08.py`)과 동일한 구조이지만, v0.9 전용 메시지 타입(`createSurface`, `updateComponents`)과 v0.9 경로 휴리스틱(상대 경로를 수정하지 않음, 절대 경로도 그대로 통과)을 검증한다. v0.9 스키마는 `oneOf`/`$defs`/`allOf` 참조 기반의 더 엄격한 JSON Schema 구조를 사용한다.

## 의존성

### 외부 패키지
- `json` (표준 라이브러리)
- `copy` (표준 라이브러리)
- `unittest.mock.MagicMock`
- `pytest`

### 저장소 내부 모듈
- [`agent_sdks/python/a2ui_agent/src/a2ui/schema/constants.py`](../../src/a2ui/schema/constants.py.md) — `A2UI_OPEN_TAG`, `A2UI_CLOSE_TAG`, `VERSION_0_9`, `SURFACE_ID_KEY`, `CATALOG_COMPONENTS_KEY`
- [`agent_sdks/python/a2ui_agent/src/a2ui/parser/constants.py`](../../src/a2ui/parser/constants.py.md) — `MSG_TYPE_CREATE_SURFACE`, `MSG_TYPE_UPDATE_COMPONENTS`, `MSG_TYPE_DELETE_SURFACE`, `MSG_TYPE_DATA_MODEL_UPDATE`
- [`agent_sdks/python/a2ui_agent/src/a2ui/schema/catalog.py`](../../src/a2ui/schema/catalog.py.md) — `A2uiCatalog`
- [`agent_sdks/python/a2ui_agent/src/a2ui/parser/streaming.py`](../../src/a2ui/parser/streaming.py.md) — `A2uiStreamParser`
- [`agent_sdks/python/a2ui_agent/src/a2ui/parser/response_part.py`](../../src/a2ui/parser/response_part.py.md) — `ResponsePart`

## Exports

pytest에 의해 자동 수집되는 테스트 함수와 헬퍼들이 정의되어 있으며 명시적 `__all__`은 없다.

| 이름 | 종류 |
|------|------|
| `mock_catalog` | pytest 픽스처 (함수) |
| `_normalize_messages` | 내부 헬퍼 함수 |
| `assertResponseContainsMessages` | 헬퍼 함수 |
| `assertResponseContainsNoA2UI` | 헬퍼 함수 |
| `assertResponseContainsText` | 헬퍼 함수 |
| `test_add_msg_type_deduplication` | 테스트 함수 |
| `test_streaming_msg_type_deduplication` | 테스트 함수 |
| `test_v09_path_heuristic_relative_path` | 테스트 함수 |
| `test_v09_path_heuristic_absolute_path` | 테스트 함수 |

## 테스트 케이스 명세

### 픽스처: `mock_catalog`

**종류:** `@pytest.fixture`

v0.9 스키마 구조를 가지는 `A2uiCatalog` 인스턴스를 반환한다. 세 개의 딕셔너리로 구성된다:

- **`s2c_schema`**: `"$schema": "https://json-schema.org/draft/2020-12/schema"`, `"$id": "https://a2ui.org/specification/v0_9/server_to_client.json"`, `"title": "A2UI Message Schema"`. 최상위에 `"oneOf"` 배열로 `CreateSurfaceMessage`, `UpdateComponentsMessage`, `UpdateDataModelMessage`, `DeleteSurfaceMessage` 네 가지 메시지 타입을 `$defs`로 정의하고 참조한다. 각 메시지 타입은 최상위에 `"version": {"const": "v0.9"}` 필드를 필수로 요구하며 `"additionalProperties": False`이다. `UpdateComponentsMessage`의 `components`는 `catalog.json#/$defs/anyComponent`를 참조한다. `UpdateDataModelMessage`는 `"path"` 없이 `"value"`만 `additionalProperties: True`로 허용한다.

- **`catalog_schema`**: `"catalogId": "test_catalog"`. `components`에 `Container`, `Card`, `Text`, `Column`, `AudioPlayer`, `List`, `Row` 컴포넌트를 정의한다. 각 컴포넌트는 `"allOf"` 배열 안에 `common_types.json#/$defs/ComponentCommon`과 `#/$defs/CatalogComponentCommon` 참조를 포함하며, `"component"` 필드에 `{"const": "컴포넌트명"}` 형태로 타입을 고정한다. `$defs`에 `CatalogComponentCommon`(선택적 `weight: number`)과 `anyComponent`(`oneOf` + `discriminator: {propertyName: "component"}`)를 정의한다.

- **`common_types_schema`**: `"$schema": "https://json-schema.org/draft/2020-12/schema"`, `"$id": "https://a2ui.org/specification/v0_9/common_types.json"`. `$defs`에 `ComponentId` (string), `AccessibilityAttributes` (label: DynamicString), `Action` (additionalProperties: True), `ComponentCommon` (id: ComponentId, required), `DataBinding` (object), `DynamicString` (anyOf: string | DataBinding), `DynamicValue` (anyOf: object | array | DataBinding), `DynamicNumber` (anyOf: number | DataBinding), `ChildList` (oneOf: ComponentId 배열 | {componentId, path} 객체)를 정의한다.

`A2uiCatalog(version=VERSION_0_9, name="test_catalog", s2c_schema=..., common_types_schema=..., catalog_schema=...)`를 반환한다.

---

### 헬퍼: `_normalize_messages(messages) -> list`

v0.8 버전의 동일 함수와 구조가 같으나, `MSG_TYPE_SURFACE_UPDATE` 대신 `MSG_TYPE_UPDATE_COMPONENTS`를 기준으로 `CATALOG_COMPONENTS_KEY` 배열을 `id` 기준 정렬한다. `ResponsePart` 인스턴스에서는 `a2ui_json`을 추출하며 `copy.deepcopy`를 사용한다.

---

### 헬퍼: `assertResponseContainsMessages(response, expected_messages) -> None`

`_normalize_messages(response) == _normalize_messages(expected_messages)` 어서션을 수행한다.

---

### 헬퍼: `assertResponseContainsNoA2UI(response) -> None`

`len(response) == 0` 이거나 `response[0].a2ui_json == None`임을 어서션한다.

---

### 헬퍼: `assertResponseContainsText(response, expected_text) -> None`

`response`의 각 항목이 `ResponsePart`이면 `p.text`를, 아니면 `p` 자체를 `expected_text`와 비교하여, 하나 이상 일치하는 항목이 있는지 어서션한다.

---

### 테스트: `test_add_msg_type_deduplication`

**픽스처/모킹:** 없음

**검증 동작:** `A2uiStreamParser()` (카탈로그 없이 생성)의 `add_msg_type`가 중복 타입을 추가하지 않는지 확인한다.

1. `MSG_TYPE_UPDATE_COMPONENTS`를 두 번 추가한 뒤 `[MSG_TYPE_UPDATE_COMPONENTS]`임을 확인한다.
2. `MSG_TYPE_CREATE_SURFACE`를 추가한 뒤 `[MSG_TYPE_UPDATE_COMPONENTS, MSG_TYPE_CREATE_SURFACE]`임을 확인한다.
3. `MSG_TYPE_UPDATE_COMPONENTS`를 다시 추가해도 목록이 변하지 않음을 확인한다.

---

### 테스트: `test_streaming_msg_type_deduplication(mock_catalog)`

**픽스처:** `mock_catalog`

**검증 동작:** 실제 v0.9 형식 청크를 파싱하는 동안 `msg_types`에 중복이 생기지 않고, 파싱 완료 후 초기화되는지 확인한다.

1. `A2uiStreamParser(catalog=mock_catalog)` 를 생성한다.
2. 첫 번째 청크: `A2UI_OPEN_TAG + '[{"version": "v0.9", "updateComponents": {"surfaceId": "s1", "root": "root", "components": [{"id": "root", "component": "Text", "text": "Hello"}'` — 아직 JSON이 완결되지 않은 상태. 스니핑이 동작해 `MSG_TYPE_UPDATE_COMPONENTS`가 `msg_types`에 정확히 한 번 포함되어야 한다.
3. 두 번째 청크: `', {"id": "c1", "component": "Text", "text": "hi"}]}} ' + A2UI_CLOSE_TAG` — JSON 완결. 파싱 완료 후 `parser.msg_types`가 비어있어야 한다(`not parser.msg_types`).

---

### 테스트: `test_v09_path_heuristic_relative_path(mock_catalog)`

**픽스처:** `mock_catalog`

**검증 동작:** v0.9 파서가 `{"path": "some/relative/path"}` 형식의 상대 경로를 수정하지 않고 그대로 통과시키는지 확인한다.

1. `parser._validator = None`으로 검증을 비활성화한다.
2. `createSurface` 메시지 청크를 처리한다 (`surfaceId = "s1"`, `catalogId = "c1"`).
3. `updateComponents` 메시지 청크를 처리한다. 컴포넌트는 `{"id": "root", "component": "Text", "text": {"path": "some/relative/path"}}` 형식이다.
4. 방출된 파트에서 `a2ui_json`을 수집하고, 첫 번째 `updateComponents` 메시지의 첫 번째 컴포넌트에서 `text.path` 값이 `"some/relative/path"`(슬래시 미추가)임을 어서션한다.

---

### 테스트: `test_v09_path_heuristic_absolute_path(mock_catalog)`

**픽스처:** `mock_catalog`

**검증 동작:** v0.9 파서가 `{"path": "/absolute/path"}` 형식의 절대 경로도 그대로 통과시키는지 확인한다.

1. `parser._validator = None`으로 검증을 비활성화한다.
2. `createSurface` 메시지 청크를 처리한다 (`surfaceId = "s1"`, `catalogId = "c1"`).
3. `updateComponents` 메시지 청크를 처리한다. 컴포넌트는 `{"id": "root", "component": "Text", "text": {"path": "/absolute/path"}}` 형식이다.
4. 방출된 파트에서 `a2ui_json`을 수집하고, 첫 번째 `updateComponents` 메시지의 첫 번째 컴포넌트에서 `text.path` 값이 `"/absolute/path"`임을 어서션한다.

## 동작 흐름

파일 로드 시 `mock_catalog` 픽스처가 정의되고, 네 개의 테스트 함수가 pytest에 의해 수집된다. 각 테스트는 독립적으로 실행된다. `test_streaming_msg_type_deduplication`, `test_v09_path_heuristic_relative_path`, `test_v09_path_heuristic_absolute_path`는 `mock_catalog` 픽스처를 주입받는다. 경로 휴리스틱 테스트 두 개는 동일한 패턴으로 `createSurface` → `updateComponents` 순서의 청크를 전달하며, v0.9에서는 v0.8과 달리 경로 변환이 발생하지 않음을 검증한다. `_normalize_messages`와 `assert*` 헬퍼는 향후 테스트 확장을 위해 정의되어 있다.
