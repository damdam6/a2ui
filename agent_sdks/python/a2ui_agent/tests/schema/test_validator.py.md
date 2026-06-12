# agent_sdks/python/a2ui_agent/tests/schema/test_validator.py

## 개요

`a2ui.schema.validator` 모듈의 핵심 함수들과 `A2uiCatalog.validator` 공개 인터페이스를 검증하는 pytest 테스트 클래스(`TestValidator`)를 담고 있다. v0.8과 v0.9 두 버전의 카탈로그를 픽스처로 구축해 스키마 번들링, 오류 메시지 품질, 컴포넌트 참조 필드 추출, 토폴로지 분석(순환 참조/자기 참조/도달 가능성) 등을 포괄적으로 검증한다.

## 의존성

### 외부 패키지
- `json` (표준 라이브러리)
- `copy` (표준 라이브러리)
- `pytest`
- `unittest.mock` (`MagicMock`)

### 저장소 내부 모듈
- [`a2ui.schema.manager`](../../src/a2ui/schema/manager.py.md) — `A2uiSchemaManager`, `A2uiCatalog`, `CatalogConfig`
- [`a2ui.schema.common_modifiers`](../../src/a2ui/schema/common_modifiers.py.md) — `remove_strict_validation`
- [`a2ui.schema.constants`](../../src/a2ui/schema/constants.py.md) — `VERSION_0_8`, `VERSION_0_9`
- [`a2ui.schema.validator`](../../src/a2ui/schema/validator.py.md) — `_find_root_id`(as `find_root_id`), `extract_component_ref_fields`, `analyze_topology`, `get_component_references`

## Exports

테스트 클래스와 픽스처만 정의하며 외부에 공개하는 심볼은 없다.

## 테스트 케이스 상세

### 클래스: `TestValidator`

모든 테스트 케이스를 하나의 클래스 내에 집약한다.

---

#### 픽스처: `catalog_0_9(self)`

- **반환 타입**: `A2uiCatalog`
- **구성 데이터**:
  - `s2c_schema`: `$id`가 `"https://a2ui.org/specification/v0_9/server_to_client.json"`. 최상위 `oneOf`로 `CreateSurfaceMessage`, `UpdateComponentsMessage`, `UpdateDataModelMessage`, `DeleteSurfaceMessage` 네 가지 메시지 타입을 정의하는 JSON Schema 딕셔너리. 각 메시지는 `"version": {"const": "v0.9"}`와 해당 메시지 키를 `required`로 가지며 `additionalProperties: False`로 닫혀 있다. `UpdateComponentsMessage`의 `components` 배열 items는 `{"$ref": "catalog.json#/$defs/anyComponent"}`를 참조한다.
  - `catalog_schema`: `$id`가 `"https://a2ui.org/specification/v0_9/catalogs/basic/catalog.json"`, `catalogId`가 `"https://a2ui.dev/specification/v0_9/catalogs/basic/catalog.json"`. `components`로 `Text`, `Image`, `Icon`, `Column`, `Card`, `Button`, `List` 7개 컴포넌트를 정의. 각 컴포넌트는 `allOf`로 `ComponentCommon`과 `CatalogComponentCommon`을 상속하고 `unevaluatedProperties: False`. `$defs`에 `CatalogComponentCommon`(`weight` 숫자 속성)과 `anyComponent`(`oneOf` 7가지 + `discriminator.propertyName: "component"`) 포함.
  - `common_types_schema`: `$id`가 `"https://a2ui.org/specification/v0_9/common_types.json"`. `$defs`에 `ComponentId`(문자열), `AccessibilityAttributes`, `Action`(자유 객체), `ComponentCommon`(`id` 필드 required), `DataBinding`(빈 객체), `DynamicString`(`string | DataBinding`), `DynamicValue`(`object | array | DataBinding`), `DynamicNumber`(`number | DataBinding`), `ChildList`(`ComponentId` 배열 또는 `{componentId, path}` 객체의 `oneOf`) 정의.
  - `A2uiCatalog(version=VERSION_0_9, name="standard", catalog_schema=..., s2c_schema=..., common_types_schema=...)` 로 인스턴스 생성.

#### 픽스처: `catalog_0_8(self)`

- **반환 타입**: `A2uiCatalog`
- **구성 데이터**:
  - `s2c_schema`: v0.8 스타일. 최상위가 `type: object`, `additionalProperties: False`. `properties`에 `beginRendering`(`surfaceId` required, `root` 및 `styles` 선택), `surfaceUpdate`(`surfaceId`와 `components` required, 각 컴포넌트는 `{id, component}` 구조), `dataModelUpdate`(`surfaceId`, `contents`) 포함.
  - `catalog_schema`: `catalogId`가 `"https://a2ui.org/specification/v0_8/json/standard_catalog_definition.json"`. `components`에 `Column`(children 배열), `Card`(child 문자열), `Button`(label, action.functionCall), `Text`(text: string|object), `List`(자유) 정의. 모두 `additionalProperties: True`. `styles`에 `font`, `primaryColor` 정의.
  - `A2uiCatalog(version=VERSION_0_8, name="standard", catalog_schema=..., s2c_schema=..., common_types_schema=None)` 으로 인스턴스 생성.

#### 픽스처: `test_catalog(self, request, catalog_0_8, catalog_0_9)`

- **파라미터화**: `@pytest.fixture(params=[VERSION_0_8, VERSION_0_9])`
- **동작**: `request.param`이 `VERSION_0_8`이면 `catalog_0_8`, 그 외면 `catalog_0_9`를 반환. 이를 통해 동일 테스트를 두 버전에 대해 자동으로 반복 실행할 수 있다.

---

#### `test_pretty_error_messages(self, catalog_0_9)`

- **검증 동작**: `catalog_0_9.validator.validate(payload)`가 복잡한 유효성 오류를 담은 페이로드에 대해 `ValueError`를 발생시키고, 오류 메시지 문자열에 사람이 읽기 쉬운 구체적인 설명이 포함되는지 확인한다.
- **페이로드 구성** (5개 메시지):
  1. 정상 `createSurface` 메시지.
  2. `updateComponents` 메시지 — `Column`에 미지원 속성 `"gap"`, `Image`에 `"altText"`·`"fit"` (미지원), 알 수 없는 컴포넌트 `"Row"`, `Text`에 미지원 속성 `"usageHint"`, `Image.url`에 문자열 대신 객체 `{"path": "/image"}`.
  3. 정상 `updateDataModel` 메시지.
  4. `deleteSurface` 메시지 — `deleteSurface` 키가 빈 객체(`{}`)여서 `surfaceId` 누락.
  5. 알 수 없는 메시지 타입 `{"unknownMessage": {}}`.
- **단언** (오류 메시지 내용에 모두 포함되어야 함):
  - `"Unknown component: Row"`.
  - `"'usageHint' was unexpected"`.
  - `"'gap' was unexpected"`.
  - `"'altText', 'fit' were unexpected"`.
  - `"'surfaceId' is a required property"`.
  - `"{'path': '/image'} is not of type 'string'"`.
  - `"Unknown message type with keys ['unknownMessage']"`.

#### `test_bundle_0_8(self, catalog_0_8)`

- **검증 동작**: `catalog_0_8.validator._bundle_0_8_schemas()`가 반환하는 번들 딕셔너리에 카탈로그의 `styles`와 `components` 정보가 올바르게 주입되었는지 확인한다.
- **단언**:
  - `bundled["properties"]["beginRendering"]["properties"]["styles"]["additionalProperties"] is False`.
  - `"font"`, `"primaryColor"`가 styles의 `properties`에 존재.
  - `bundled["properties"]["surfaceUpdate"]["properties"]["components"]["items"]["properties"]["component"]["additionalProperties"] is False`.
  - `"Text"`, `"Button"`이 component의 `properties`에 존재.
- **픽스처**: `catalog_0_8`.

#### `test_find_root_id_v08(self)`

- **검증 동작**: v0.8 스타일 메시지 목록에서 `find_root_id`(`_find_root_id`) 함수가 `beginRendering.root` 값을 올바르게 추출하는지 확인한다.
- **입력**: `[{"beginRendering": {"surfaceId": "s1", "root": "custom-root"}}, {"surfaceUpdate": ...}]`.
- **단언**: `find_root_id(messages) == "custom-root"`.
- **픽스처/모킹**: 없음.

#### `test_find_root_id_v09(self)`

- **검증 동작**: v0.9 스타일 메시지 목록에서 `find_root_id`가 올바른 루트 ID를 반환하는지 두 가지 케이스를 확인한다.
  - `createSurface` 메시지가 있을 때 → `"root"` 반환.
  - `updateComponents`만 있는 증분 업데이트의 경우(루트 없음) → `None` 반환.
- **픽스처/모킹**: 없음.

#### `test_get_component_references(self)`

- **검증 동작**: `get_component_references` 함수가 `ref_fields_map`을 기반으로 컴포넌트 딕셔너리에서 단일 참조(`child`)와 목록 참조(`children`)를 올바르게 추출하는지 확인한다.
- **입력**:
  - `ref_fields_map = {"Container": ({"child"}, {"children"}), "Text": (set(), set())}`.
  - `comp = {"id": "c1", "component": "Container", "child": "c2", "children": ["c3", "c4"]}`.
- **단언**: 결과 리스트에 `("c2", "child")`, `("c3", "children")`, `("c4", "children")` 모두 포함.
- **픽스처/모킹**: 없음.

#### `test_analyze_topology_circular(self)`

- **검증 동작**: 순환 참조(`c1 → c2 → c1`)가 있는 컴포넌트 집합에 대해 `analyze_topology`가 `ValueError("Circular reference detected")`를 발생시키는지 확인한다.
- **입력**: `ref_fields_map = {"Node": ({"next"}, set())}`, `components = [{"id": "c1", "component": "Node", "next": "c2"}, {"id": "c2", "component": "Node", "next": "c1"}]`.
- **픽스처/모킹**: 없음.

#### `test_analyze_topology_self_ref(self)`

- **검증 동작**: 자기 자신을 참조하는(`c1 → c1`) 컴포넌트에 대해 `analyze_topology`가 `ValueError("Self-reference detected")`를 발생시키는지 확인한다.
- **입력**: `components = [{"id": "c1", "component": "Node", "next": "c1"}]`.
- **픽스처/모킹**: 없음.

#### `test_analyze_topology_reachable(self)`

- **검증 동작**: `analyze_topology`가 루트로부터 순회를 통해 도달 가능한 컴포넌트 집합만 올바르게 반환하는지 확인한다. 고립된(`"orphan"`) 컴포넌트는 결과에서 제외된다.
- **입력**: `root="root"`, 4개 컴포넌트(`root → c1 → c2`, `orphan`은 참조 없음).
- **단언**: `reachable == {"root", "c1", "c2"}`.
- **픽스처/모킹**: 없음.

#### `test_extract_component_ref_fields_mock(self)`

- **검증 동작**: `extract_component_ref_fields` 함수가 `A2uiCatalog`의 `catalog_schema.components` 속성 정의를 분석해 단일 참조 필드(`ComponentId`를 직접 참조)와 목록 참조 필드(`ChildList`를 참조)를 올바르게 구별해 `ref_fields_map`을 생성하는지 확인한다.
- **모킹**: `MagicMock(spec=A2uiCatalog)`로 카탈로그를 흉내낸다.
  - `catalog.version = VERSION_0_9`.
  - `catalog.common_types_schema`: `$defs`에 `ComponentId`(string)와 `ChildList`(배열 또는 `{componentId}` 객체의 `oneOf`) 정의 포함.
  - `catalog.catalog_schema`: `components`에 `MyComp` 정의. `MyComp.properties`에 `ref`(`$ref: "common_types.json#/$defs/ComponentId"`)와 `multi`(`$ref: "common_types.json#/$defs/ChildList"`) 포함.
  - `catalog.s2c_schema = {}`.
- **단언**:
  - `"MyComp" in ref_map`.
  - `"ref" in single_refs` (단일 참조 집합).
  - `"multi" in list_refs` (목록 참조 집합).

## 동작 흐름

`TestValidator` 클래스는 두 개의 풍부한 픽스처(`catalog_0_8`, `catalog_0_9`)를 바탕으로 테스트를 구성한다. `test_pretty_error_messages`와 `test_bundle_0_8`은 `A2uiCatalog.validator` 공개 인터페이스를 통해 검증하며, 나머지 테스트들은 `validator.py`의 내부/공개 함수들을 직접 임포트해 단위 테스트한다. 토폴로지 분석 테스트 3개는 순환/자기참조/정상 도달 가능성의 세 경계 케이스를 망라한다. `test_extract_component_ref_fields_mock`은 실제 파일 I/O 없이 스키마 구조만 mock으로 제공해 필드 분류 알고리즘을 독립 검증한다.
