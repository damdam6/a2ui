# agent_sdks/python/a2ui_agent/tests/parser/test_streaming_v08.py

## 개요

이 파일은 v0.8 프로토콜 기준으로 `A2uiStreamParser`의 동작을 검증하는 pytest 테스트 모음이다. 메시지 타입 중복 등록 방지(deduplication)와 스트리밍 파싱 중 발생하는 v0.8 전용 경로 휴리스틱(상대 경로에 자동으로 선행 슬래시 추가) 동작을 테스트한다. 인메모리 mock catalog 픽스처를 사용하여 실제 스키마 파일 없이도 파서 동작을 검증한다.

## 의존성

### 외부 패키지
- `json` (표준 라이브러리)
- `copy` (표준 라이브러리)
- `unittest.mock.MagicMock`
- `pytest`

### 저장소 내부 모듈
- [`agent_sdks/python/a2ui_agent/src/a2ui/schema/constants.py`](../../src/a2ui/schema/constants.py.md) — `A2UI_OPEN_TAG`, `A2UI_CLOSE_TAG`, `VERSION_0_8`, `SURFACE_ID_KEY`, `CATALOG_COMPONENTS_KEY`
- [`agent_sdks/python/a2ui_agent/src/a2ui/parser/constants.py`](../../src/a2ui/parser/constants.py.md) — `MSG_TYPE_SURFACE_UPDATE`, `MSG_TYPE_BEGIN_RENDERING`, `MSG_TYPE_DELETE_SURFACE`, `MSG_TYPE_DATA_MODEL_UPDATE`
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
| `test_v08_path_heuristic_adds_slash` | 테스트 함수 |

## 테스트 케이스 명세

### 픽스처: `mock_catalog`

**종류:** `@pytest.fixture`

v0.8 스키마 구조를 가지는 `A2uiCatalog` 인스턴스를 반환한다. 세 개의 딕셔너리로 구성된다:

- **`s2c_schema`**: `"title": "A2UI Message Schema"`, `"type": "object"`, `"additionalProperties": False`. `properties`에 `beginRendering`, `surfaceUpdate`, `dataModelUpdate`, `deleteSurface` 네 가지 메시지 타입을 정의한다. 각 메시지 타입은 `surfaceId`를 `required` 필드로 가지며, `surfaceUpdate`는 추가로 `components` 배열(각 항목에 `id`와 `component` 필드를 요구)을 필수로 요구한다. `dataModelUpdate`는 `contents` 배열을 필수로 요구하며, 각 항목은 `key`와 선택적 `valueString`, `valueNumber`, `valueBoolean`, `valueMap`을 가진다.
- **`catalog_schema`**: `"catalogId": "test_catalog"`. `components`에 `Container`, `Card`, `Text`, `Loading`, `List`, `Row`, `Column`, `AudioPlayer` 컴포넌트를 정의한다. `Row`와 `Column`은 `children.explicitList` 구조를 가지며, `AudioPlayer`는 `url`을 필수로 요구한다.
- **`common_types_schema`**: `"$id": "https://a2ui.org/specification/v0_8/common_types.json"`, `"type": "object"`, 빈 `"$defs": {}`.

`A2uiCatalog(version=VERSION_0_8, name="test_catalog", s2c_schema=..., common_types_schema=..., catalog_schema=...)`를 반환한다.

---

### 헬퍼: `_normalize_messages(messages) -> list`

**매개변수:** `messages` — `ResponsePart` 또는 딕셔너리의 리스트

메시지 목록을 안정적인 비교를 위해 정규화한다. `ResponsePart` 인스턴스인 경우 `m.a2ui_json`을 추출하되, `a2ui_json`이 리스트이면 `extend`, 아니면 `append`로 결과에 추가한다. 모든 원소는 `copy.deepcopy`로 복사한다. 이후 결과 내 각 메시지에서 `MSG_TYPE_SURFACE_UPDATE` 키를 찾고, 그 안의 `CATALOG_COMPONENTS_KEY` 배열을 `id` 필드 기준 오름차순으로 정렬한다. 정렬된 리스트를 반환한다.

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

**검증 동작:** `A2uiStreamParser()` (카탈로그 없이 생성)의 `add_msg_type` 메서드가 중복 타입을 추가하지 않는지 확인한다.

1. `MSG_TYPE_SURFACE_UPDATE`를 두 번 추가한 뒤 `parser.msg_types == [MSG_TYPE_SURFACE_UPDATE]`임을 확인한다.
2. `MSG_TYPE_BEGIN_RENDERING`을 추가한 뒤 `[MSG_TYPE_SURFACE_UPDATE, MSG_TYPE_BEGIN_RENDERING]`임을 확인한다.
3. `MSG_TYPE_SURFACE_UPDATE`를 다시 추가해도 목록이 변하지 않음을 확인한다.

---

### 테스트: `test_streaming_msg_type_deduplication(mock_catalog)`

**픽스처:** `mock_catalog`

**검증 동작:** 실제 청크를 파싱하는 과정에서도 `msg_types`에 중복이 추가되지 않는지, 그리고 파싱 완료 후 `msg_types`가 초기화되는지 확인한다.

1. `A2uiStreamParser(catalog=mock_catalog)` 를 생성한다.
2. `A2UI_OPEN_TAG + '[{"surfaceUpdate": {"surfaceId": "s1", "components": ['` 를 첫 번째 청크로 전달한다. 스니핑이 동작해 `MSG_TYPE_SURFACE_UPDATE`가 `msg_types`에 한 번만 존재해야 한다.
3. 나머지 JSON과 `A2UI_CLOSE_TAG`를 두 번째 청크로 전달한다. 파싱 완료 후 `parser.msg_types == []`임을 확인한다.

---

### 테스트: `test_v08_path_heuristic_adds_slash(mock_catalog)`

**픽스처:** `mock_catalog`

**검증 동작:** v0.8 파서가 컴포넌트 속성의 `{"path": "some/relative/path"}` 값에 자동으로 선행 슬래시를 추가하는지 확인한다.

1. `A2uiStreamParser(catalog=mock_catalog)` 를 생성하고 `parser._validator = None`으로 검증을 비활성화한다.
2. `beginRendering` 메시지를 담은 첫 번째 청크를 처리한다 (`surfaceId = "s1"`, `root = "root"`). 이는 이후 `surfaceUpdate`가 버퍼링 없이 즉시 방출되도록 하는 전처리 역할을 한다.
3. `surfaceUpdate` 메시지를 담은 두 번째 청크를 처리한다. 컴포넌트는 `{"id": "root", "component": {"Text": {"text": {"path": "some/relative/path"}}}}` 형식이다.
4. 방출된 파트에서 `a2ui_json`을 수집하고, 첫 번째 메시지의 컴포넌트에서 `path` 값이 `"/some/relative/path"`(선행 슬래시 포함)인지 어서션한다.

## 동작 흐름

파일 로드 시 `mock_catalog` 픽스처가 정의되고, 세 개의 테스트 함수가 pytest에 의해 수집된다. 각 테스트는 독립적으로 실행된다. `test_streaming_msg_type_deduplication`과 `test_v08_path_heuristic_adds_slash`는 `mock_catalog` 픽스처를 주입받아 실제 스트림 파싱 흐름을 시뮬레이션한다. `_normalize_messages`와 세 개의 `assert*` 헬퍼는 현재 테스트에서 직접 호출되지 않지만, 같은 디렉토리의 다른 테스트 파일에서 재사용되거나 향후 테스트 확장을 위해 정의되어 있다.
