# agent_sdks/python/a2ui_core/tests/test_catalog.py

## 개요

`CatalogApi` 인터페이스를 구현하는 세 가지 카탈로그 클래스(`ModelCatalog`, `JsonCatalog`, `BasicCatalog`)의 동작을 검증하는 pytest 테스트 파일이다. 각 구현체에 대해 초기화, 컴포넌트 유효성 검사, 테마 유효성 검사, 함수 유효성 검사, 레퍼런스 필드 추출 등의 핵심 기능을 커버한다. `CatalogValidator`를 통해 카탈로그 인스턴스를 간접적으로 테스트하며, 실제 사양 파일(`specification/v0_9/catalogs/basic/catalog.json`)을 로드하는 통합 테스트도 포함한다.

## 의존성

### 외부 패키지
- `typing` — `Any`, `Dict`, `List`, `Literal`, `Optional`, `Set`, `Tuple`
- `pydantic` — `BaseModel`, `Field`, `ValidationError`
- `pytest`
- `json`, `pathlib.Path` (표준 라이브러리, `test_json_catalog_basic_spec_tabs_ref` 내부 인라인 import)

### 저장소 내부 모듈
- [`agent_sdks/python/a2ui_core/src/a2ui/core/catalog/__init__.py`](../src/a2ui/core/catalog/__init__.py.md) — `CatalogApi`, `JsonCatalog`, `ModelCatalog`
- [`agent_sdks/python/a2ui_core/src/a2ui/core/validating/__init__.py`](../src/a2ui/core/validating/__init__.py.md) — `CatalogValidator`
- [`agent_sdks/python/a2ui_core/src/a2ui/core/basic_catalog/__init__.py`](../src/a2ui/core/basic_catalog/__init__.py.md) — `BasicCatalog`
- [`agent_sdks/python/a2ui_core/src/a2ui/core/schema/common_types.py`](../src/a2ui/core/schema/common_types.py.md) — `ComponentId`
- [`agent_sdks/python/a2ui_core/src/a2ui/core/schema/constants.py`](../src/a2ui/core/schema/constants.py.md) — `SPEC_VERSION`

## Exports

이 파일은 테스트 함수만 정의하며, pytest가 수집·실행한다. 모듈 수준에서 외부로 export되는 공개 심벌은 없다.

## 상세 명세

### 헬퍼 함수

#### `_val(catalog: CatalogApi) -> CatalogValidator`
`CatalogValidator.from_catalog(catalog)`를 호출하여 `CatalogValidator` 인스턴스를 반환하는 단순 팩토리 헬퍼. 모든 테스트 케이스에서 카탈로그 인스턴스로부터 검증기를 생성하는 데 사용된다.

---

### 섹션 1: ModelCatalog 테스트

#### `test_model_catalog_initialization()`
빈 pydantic `BaseModel`(`EmptyModel`)을 컴포넌트로 등록하여 `ModelCatalog(spec_version=SPEC_VERSION, catalog_id="https://a2ui.org/model-init", components={"Empty": EmptyModel})`를 생성한 뒤, `spec_version`과 `catalog_id` 속성이 생성자에 전달한 값과 정확히 일치하는지 확인한다.
- **픽스처/모킹:** 인라인 `class EmptyModel(BaseModel): pass` 정의.

#### `test_model_catalog_additional_properties_handling()`
세 가지 모델을 가진 `ModelCatalog`로 `extra` 설정별 추가 프로퍼티 처리를 검증한다:
- `DefaultBox`: `extra` 미설정 (기본값). `extraProp` 포함 payload가 통과해야 한다.
- `AllowBox`: `model_config = {"extra": "allow"}`. `extraProp` 포함 payload가 통과해야 한다.
- `ForbidBox`: `model_config = {"extra": "forbid"}`. `extraProp` 포함 payload는 `ValidationError`를 발생시켜야 하며, 오류 메시지에 `"extraProp"` 또는 `"Extra inputs"`가 포함되어야 한다.
- **픽스처/모킹:** 각 모델 인라인 정의, `catalog_id="https://a2ui.org/model-extra"`.

#### `test_model_catalog_unevaluated_properties_handling()`
`json_schema_extra`를 통한 JSON Schema 수준 `unevaluatedProperties` 제어를 테스트한다:
- `DefaultBox`: 기본 설정. 추가 속성을 허용해야 한다.
- `AllowBox`: `model_config = {"json_schema_extra": {"unevaluatedProperties": True}}`. 추가 속성을 허용해야 한다.
- `ForbidBox`: `model_config = {"json_schema_extra": {"unevaluatedProperties": False}}`. `extraProp` 포함 시 `ValidationError` 또는 `ValueError`(메시지에 `"extraProp"` 또는 `"Extra inputs"` 포함)를 발생시켜야 한다.
- **픽스처/모킹:** `catalog_id="https://a2ui.org/model-unevaluated"`, 세 모델 인라인 정의.

#### `test_model_catalog_validate_theme()`
`primary` 필드에 `Field(..., pattern="^#[0-9A-F]{6}$")` 제약이 있는 `TestTheme`을 `theme=TestTheme`으로 등록한 카탈로그를 구성한다:
1. `{"primary": "#00FF00"}` — `validate_theme()` 호출이 통과해야 한다.
2. `{"primary": "blue"}` — `ValidationError`가 발생해야 하며, 오류 메시지에 `"primary"` 및 (`"pattern"` 또는 `"string"`)이 포함되어야 한다.
- **픽스처/모킹:** `catalog_id="https://a2ui.org/model"`, `TestTheme(BaseModel)` 인라인 정의.

#### `test_model_catalog_nested_function_validation()`
`InnerComp`(call: str, args: `Dict[str, Any]`) 필드를 가진 `OuterComp`와 `CustomFunc`(call: `Literal["custom"]`, args: `CustomFuncArgs(param: int)`)가 등록된 카탈로그로 세 시나리오를 검증한다:
1. `inner: {"call": "custom", "args": {"param": 123}}` — 통과.
2. `inner: {"call": "unrecognizedFunctionName", "args": {}}` — `ValueError("Unknown function: unrecognizedFunctionName")` 발생.
3. `inner: {"call": "custom", "args": {"param": "not-an-int"}}` — `ValueError("Invalid function call 'custom'")` 발생.
- **픽스처/모킹:** `InnerComp`, `OuterComp`, `CustomFuncArgs`, `CustomFunc` 인라인 정의.

#### `test_model_catalog_validate_components()`
`ButtonComp`(id: str, component: `Literal["Button"]`, label: str)를 등록한 카탈로그로:
1. `{"id": "b1", "component": "Button", "label": "Click"}` — 통과.
2. `{"id": "b1", "component": "Button"}` (label 누락) — `ValidationError`(메시지에 `"label"` 및 `"Field required"` 또는 `"missing"` 포함) 발생.
- **픽스처/모킹:** `ButtonComp(BaseModel)` 인라인 정의.

#### `test_model_catalog_validate_functions()`
`RegexFunc`(call: `Literal["regex"]`, args: `Dict[str, Any]`)가 등록된 카탈로그로:
1. `validate_function("regex", {"pattern": "^[A-Z]+$"})` — 통과.
2. `validate_function("unknownFunc", {})` — `ValueError("Unknown function")` 발생.
- **픽스처/모킹:** `RegexFunc(BaseModel)` 인라인 정의.

#### `test_model_catalog_custom_reference_fields_coverage()`
`primaryPtr: ComponentId`(단일 참조)와 `secondaryPtrs: List[ComponentId]`(목록 참조)를 가진 `CustomLayoutComp`를 등록한 카탈로그로 세 가지 검증:
1. `catalog.extract_ref_fields()`가 `"CustomLayout"` 키 아래 첫 번째 원소(단일 참조 set)에 `"primaryPtr"`, 두 번째 원소(목록 참조 set)에 `"secondaryPtrs"`를 포함해야 한다.
2. 5개 노드(root → nodeA/nodeB → leaf → end → root 순환)로 구성된 페이로드가 `validate_components()`를 통과해야 한다.
3. 고아 노드(`orphanNode`)가 포함된 페이로드도 스키마 검사가 위상 정렬 기반이 아니므로 통과해야 한다.
- **픽스처/모킹:** `catalog_id="https://a2ui.org/custom"`, `CustomLayoutComp(BaseModel)` 인라인 정의.

#### `test_model_catalog_unrecognized_type_and_mismatched_properties()`
`CardComp`(id: str, component: `Literal["Card"]`, elevation: int, `model_config = {"extra": "forbid"}`)를 등록한 카탈로그로:
1. `{"id": "c1", "component": "NonExistent"}` — `ValueError("Unknown component type: NonExistent")` 발생.
2. `{"id": "c1", "component": "Card", "elevation": 1, "extraProperty": "garbage"}` — `ValidationError`(메시지에 `"extra_forbidden"` 또는 `"extra"` 포함) 발생.
3. `{"id": "c1", "component": "Card", "elevation": "high"}` — `ValidationError`(메시지에 `"int_parsing"` 또는 `"integer"` 포함) 발생.
- **픽스처/모킹:** `catalog_id="https://a2ui.org/model-extended"`, `CardComp(BaseModel)` 인라인 정의.

---

### 섹션 2: JsonCatalog 테스트

#### `test_json_catalog_initialization()`
`"catalogId": "https://a2ui.org/spec/v0.9/catalog.json"`이 포함된 JSON schema로 `JsonCatalog(spec_version=SPEC_VERSION, catalog_schema=schema)`를 생성하고, `catalog.catalog_id`가 schema의 `"catalogId"` 값과 일치하는지 확인한다.
- **픽스처/모킹:** `"Text"` 컴포넌트(`additionalProperties: False`)를 포함한 인라인 딕셔너리.

#### `test_json_catalog_extract_ref_fields_dynamic_coverage()`
`AdvancedLayout` 컴포넌트에 5가지 유형의 속성을 정의한 schema로 `extract_ref_fields()`를 테스트한다:
- `customChild`: `{"$ref": "common_types.json#/$defs/ComponentId"}` → 단일 레퍼런스에 포함.
- `customList`: `{"$ref": "common_types.json#/$defs/ChildList"}` → 목록 레퍼런스에 포함.
- `nestedChild`: `{"allOf": [{"$ref": "#/$defs/ComponentId"}]}` → 단일 레퍼런스에 포함.
- `nestedList`: `{"oneOf": [{"$ref": "#/$defs/ChildList"}]}` → 목록 레퍼런스에 포함.
- `regularProp`: `{"type": "string"}` → 어느 레퍼런스에도 포함되지 않음.

#### `test_json_catalog_extract_ref_fields_tabs()`
`Tabs` 컴포넌트의 `tabs` 속성이 `array of objects`(각 item에 `title: $ref DynamicString`, `child: $ref ComponentId` 포함)로 정의된 schema를 검사한다. 결과에서 `tabs`는 목록 레퍼런스(`list_refs`)에 포함되어야 하고, 내부 `child`는 단일 레퍼런스(`single`)에 직접 등록되지 않아야 한다.

#### `test_json_catalog_extract_ref_fields_empty_fallback()`
속성이 비어 있는 `EmptyNode` 컴포넌트를 가진 schema로 `extract_ref_fields()`를 호출하면 빈 dict `{}`를 반환해야 한다.

#### `test_json_catalog_common_types_defs_and_refs_resolution()`
`common_types_schema`(`$id: "https://a2ui.org/specification/v0_9/common_types.json"`)에 `ColorHex`(`type: string, pattern: "^#[0-9a-fA-F]{6}$"`)를 정의하고, catalog schema의 `Box` 컴포넌트가 `color: $ref "common_types.json#/$defs/ColorHex"`를 참조하는 구조로 `JsonCatalog(spec_version, catalog_schema, common_types_schema)`를 생성한 뒤 네 가지 시나리오 검증:
1. `{"id": "b1", "component": "Box", "color": "#00FF00"}` — 통과.
2. `{"id": "b1", "component": "Box", "color": "red"}` — `ValueError("does not match")` 발생.
3. `{"id": "b1", "component": "UnknownBox", "color": "#000"}` — `ValueError("Unknown component")` 발생.
4. `{"id": "b1", "component": "Box", "color": "#FFFFFF", "extraProp": 123}` — `ValueError("Additional properties are not allowed")` 발생.

#### `test_json_catalog_custom_reference_fields_coverage()`
`masterNode`(`$ref` → full URL `ComponentId`)와 `slaveNodes`(array items `$ref` → `ComponentId`)를 가진 `CustomContainer`를 정의하고, `extract_ref_fields()`가 `"masterNode"`을 단일 레퍼런스로, `"slaveNodes"`를 목록 레퍼런스로 반환하는지 확인한다.

#### `test_json_catalog_validate_theme()`
`primaryColor: string`(hex 패턴 `^#[0-9a-fA-F]{6}$`, `additionalProperties: False`)으로 정의된 theme schema를 가진 catalog로:
1. `{"primaryColor": "#00FF00"}` — 통과.
2. `{"primaryColor": "red"}` — `ValueError("is not valid under any of the given schemas|does not match")` 발생.

#### `test_json_catalog_validate_functions()`
`regex` 함수(args에 `value: string`, `pattern: string` 필수, `additionalProperties: False`)를 가진 catalog로:
1. `validate_function("regex", {"value": "Alice", "pattern": "^[a-zA-Z]+$"})` — 통과.
2. `validate_function("regex", {"value": "Alice"})` — `ValueError("is a required property|pattern")` 발생.
3. `validate_function("unknownFunc", {})` — `ValueError("Unknown function")` 발생.

#### `test_json_catalog_nested_function_validation()`
`Text` 컴포넌트(`text` 필드가 string 또는 `{call, args}` 객체 oneOf)와 `regex` 함수를 가진 catalog로:
1. `text: {"call": "unrecognizedFunctionName", "args": {}}` — `ValueError("Unknown function: unrecognizedFunctionName")` 발생.
2. `text: {"call": "regex", "args": {"value": "Alice", "unmapped": "garbage"}}` — `ValueError("Invalid function call 'regex'|pattern|Additional properties")` 발생.

#### `test_json_catalog_additional_properties_handling()`
두 가지 catalog로 `validate_component_properties()`를 통해 추가 프로퍼티 허용 여부 검증:
1. `SimpleBox`: `additionalProperties` 미설정(기본 True) → `extraProp: 123` 포함 payload 통과.
2. `FlexBox`: `additionalProperties: True` 명시 → `extraProp: 456` 포함 payload 통과.

#### `test_json_catalog_unevaluated_properties_handling()`
`unevaluatedProperties` 세 가지 설정을 `validate_component_properties()`로 검증:
1. `DefaultBox`: 미설정(기본) → 추가 필드 허용.
2. `StrictBox`: `unevaluatedProperties: False` → `ValueError("Unevaluated properties|Additional properties")` 발생.
3. `FlexBox`: `unevaluatedProperties: True` → 추가 필드 허용.

---

### 섹션 3: BasicCatalog 테스트

#### `test_basic_catalog_initialization()`
`BasicCatalog()`를 기본 인수 없이 생성하면 `spec_version == SPEC_VERSION`이고, `catalog_id`에 `"https://a2ui.org/specification"` 문자열이 포함되어야 한다.

#### `test_basic_catalog_validate_components()`
1. `{"id": "t1", "component": "Text", "text": "Hello World", "variant": "body"}` — 통과.
2. `{"id": "t2", "component": "Text", "text": 12345}` (text가 정수) — `ValidationError` 발생.

#### `test_basic_catalog_validate_theme()`
1. `{"primaryColor": "#00BFFF"}` — 통과.
2. `{"primaryColor": "invalid-color-name"}` — `ValidationError` 발생.

#### `test_basic_catalog_validate_functions()`
1. `validate_function("formatString", {"value": "Hello ${/username}"})` — 통과.
2. `validate_function("formatString", {"invalid_param": "value"})` — `ValidationError` 발생.
3. `validate_function("unknownFunc", {})` — `ValueError("Unknown function")` 발생.

#### `test_basic_catalog_nested_function_validation()`
1. `text: {"call": "unrecognizedFunctionName", "args": {}}` — `ValueError("Unknown function: unrecognizedFunctionName")` 발생.
2. `text: {"call": "formatNumber", "args": {"value": 123.45, "decimals": "invalid-string-instead-of-number"}}` — `ValueError("Invalid function call 'formatNumber'|decimal")` 발생.

#### `test_basic_catalog_extract_ref_fields()`
`BasicCatalog`의 `extract_ref_fields()` 결과에서:
- `"Button"` 키의 단일 레퍼런스 원소에 `"child"` 포함.
- `"Column"` 키의 목록 레퍼런스 원소에 `"children"` 포함.

#### `test_basic_catalog_tabs_ref()`
`BasicCatalog`의 `extract_ref_fields()` 결과에서 `"Tabs"` 키의 목록 레퍼런스 원소에 `"tabs"` 포함.

#### `test_model_catalog_tabs_ref()`
`CustomTab(BaseModel)`(title: str, child: `ComponentId`)과 `CustomTabsComponent(BaseModel)`(component: `Literal["CustomTabs"]`, tabs: `List[CustomTab]`)를 등록한 `ModelCatalog`에서, `extract_ref_fields()` 결과의 `"CustomTabs"` 목록 레퍼런스에 `"tabs"` 포함.
- **픽스처/모킹:** `catalog_id="https://a2ui.org/tabs-test"`, 두 모델 인라인 정의.

#### `test_json_catalog_custom_tabs_ref()`
인라인 `$defs`로 `ComponentId`와 `CustomTab`(title: string, child: `$ref ComponentId`)을 정의하고, `CustomTabs` 컴포넌트의 `tabs`를 `array of $ref CustomTab`으로 구성한 `JsonCatalog`에서, `extract_ref_fields()` 결과의 `"CustomTabs"` 목록 레퍼런스에 `"tabs"` 포함.
- **픽스처/모킹:** `catalog_id="https://a2ui.org/json-tabs"`, 인라인 schema 딕셔너리.

#### `test_json_catalog_basic_spec_tabs_ref()`
`Path(__file__).parent.parent.parent.parent.parent`를 `repo_root`로 계산하여 `specification/v0_9/catalogs/basic/catalog.json`을 `json.load()`로 읽고, `JsonCatalog(spec_version, catalog_schema, catalog_id="https://a2ui.org/specification/v0_9/catalogs/basic/catalog.json")`를 구성한 뒤, `extract_ref_fields()` 결과의 `"Tabs"` 목록 레퍼런스에 `"tabs"` 포함.
- **픽스처/모킹:** `json`, `pathlib.Path` 를 함수 본문 내부에서 인라인 import. 실제 파일시스템 접근.

## 동작 흐름

모든 테스트는 독립적으로 실행된다. 공유 픽스처는 없으며, 각 테스트 함수 내부에서 필요한 pydantic 모델 또는 JSON schema 딕셔너리를 인라인으로 정의하고 카탈로그 인스턴스를 생성한다. `_val()` 헬퍼로 `CatalogValidator`를 얻은 뒤 `validate_components()`, `validate_theme()`, `validate_function()`, `validate_component_properties()`, `extract_ref_fields()` 등의 메서드를 호출하여 결과를 단언하거나 특정 예외가 발생하는지 `pytest.raises()`로 확인한다. 섹션 구조는 구현체 종류(ModelCatalog → JsonCatalog → BasicCatalog) 및 탭 레퍼런스 교차 검증 순서로 구성된다.
