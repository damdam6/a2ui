# agent_sdks/python/a2ui_core/scripts/generate_schemas.py

## 개요

이 스크립트는 A2UI 사양(specification) JSON 파일들을 읽어 Python Pydantic 모델 소스 코드를 자동 생성하는 코드 생성기다. `specification/v0_9/` 디렉터리의 JSON 스키마 정의를 파싱하여 `a2ui/core/schema/`, `a2ui/core/basic_catalog/`, `a2ui/core/catalog/` 하위에 `.py` 파일들을 출력한다. 생성된 코드는 수동 편집을 금지(`Auto-generated. Do not edit manually.`)하며, 생성 후 `pyink`로 자동 포매팅된다.

## 의존성

### 외부 패키지
- `json` (표준 라이브러리)
- `os` (표준 라이브러리)
- `re` (표준 라이브러리)
- `typing`: `Any`, `Dict`, `List`
- `copy` (`generate_common_types` 내부에서 지연 import)
- `subprocess` (`main` 내부에서 지연 import)

### 저장소 내부 모듈
없음 (이 스크립트는 독립 실행 파일이며 내부 모듈을 import하지 않는다)

## Exports

모듈 수준에서 직접 export되는 공개 심볼은 없다. 스크립트는 `if __name__ == "__main__": main()` 패턴으로 실행된다.

---

## 상세 명세

### 모듈 수준 상수

| 이름 | 값 / 설명 |
|---|---|
| `SCRIPT_DIR` | `__file__`의 절대 디렉터리 경로 |
| `SPEC_VERSION` | `"v0_9"` |
| `SPEC_VERSION_DOT` | `"v0.9"` (`_`를 `.`으로 치환) |
| `SPEC_DIR` | `SCRIPT_DIR/../../../specification/v0_9`의 절대 경로 |
| `JSON_DIR` | `"json"` |
| `CATALOGS_DIR` | `"catalogs"` |
| `COMMON_TYPES_PATH` | `SPEC_DIR/json/common_types.json` |
| `CLIENT_CAPABILITIES_PATH` | `SPEC_DIR/json/client_capabilities.json` |
| `CLIENT_TO_SERVER_PATH` | `SPEC_DIR/json/client_to_server.json` |
| `BASIC_CATALOG_PATH` | `SPEC_DIR/catalogs/basic/catalog.json` |
| `SERVER_TO_CLIENT_PATH` | `SPEC_DIR/json/server_to_client.json` |
| `SCHEMA_DIR` | `SCRIPT_DIR/../src/a2ui/core/schema`의 절대 경로 |
| `BASIC_CATALOG_DIR` | `SCRIPT_DIR/../src/a2ui/core/basic_catalog`의 절대 경로 |
| `CATALOG_DIR` | `SCRIPT_DIR/../src/a2ui/core/catalog`의 절대 경로 |
| `COMMON_TYPES_OUT_PATH` | `SCHEMA_DIR/common_types.py` |
| `CONSTANTS_OUT_PATH` | `SCHEMA_DIR/constants.py` |
| `CLIENT_CAPABILITIES_OUT_PATH` | `SCHEMA_DIR/client_capabilities.py` |
| `CLIENT_TO_SERVER_OUT_PATH` | `SCHEMA_DIR/client_to_server.py` |
| `COMPONENTS_OUT_PATH` | `BASIC_CATALOG_DIR/components.py` |
| `FUNCTION_APIS_OUT_PATH` | `BASIC_CATALOG_DIR/function_apis.py` |
| `STYLES_OUT_PATH` | `BASIC_CATALOG_DIR/styles.py` |
| `SERVER_TO_CLIENT_OUT_PATH` | `SCHEMA_DIR/server_to_client.py` |
| `SCHEMA_INIT_OUT_PATH` | `SCHEMA_DIR/__init__.py` |
| `BASIC_CATALOG_INIT_OUT_PATH` | `BASIC_CATALOG_DIR/__init__.py` (경로만 정의, 실제 생성 코드 없음) |
| `CATALOG_FUNCTIONS_OUT_PATH` | `CATALOG_DIR/functions.py` |
| `CATALOG_INIT_OUT_PATH` | `CATALOG_DIR/__init__.py` (경로만 정의, 실제 생성 코드 없음) |
| `INLINE_OBJECTS` | `Dict[str, Dict[str, Any]]` — 컴포넌트 컴파일 중 발견된 인라인 객체 스키마를 누적하는 전역 레지스트리 |
| `ALLOW_INLINE_COMPILATION` | `bool`, 초기값 `False` — 인라인 객체 클래스 생성 허용 여부 플래그 |
| `FILE_HEADER` | Apache 2.0 라이선스 헤더 + `"# Auto-generated. Do not edit manually.\n"` |

---

### `map_json_type_to_python(prop_name: str, prop: Dict[str, Any]) -> str`

JSON Schema 프로퍼티 하나를 받아 해당하는 Pydantic Python 타입 문자열을 반환한다.

처리 우선순위 (위에서 아래 순서):
1. `"const"` 키가 있으면 값이 `str`이면 `Literal['<값>']`, 아니면 `Literal[<값>]`을 반환.
2. `"$ref"` 키가 있으면: `common_types.json`을 참조하면 `ref` 경로의 마지막 세그먼트를 반환; `#/`로 시작하면 마지막 세그먼트; 그 외는 `"Any"`.
3. `"oneOf"` 또는 `"anyOf"`가 있으면 각 항목을 재귀 처리 후 중복 제거, 단일 항목이면 그대로 반환, 복수면 `Union[...]`.
4. `"allOf"`가 있으면 첫 번째 항목만 재귀 처리.
5. `"type"` 필드에 따라 분기:
   - `"string"`: `"enum"`이 있으면 `Literal[...]`, 없으면 `"str"`
   - `"number"` → `"float"`, `"integer"` → `"int"`, `"boolean"` → `"bool"`
   - `"array"`: `items`를 재귀 처리하여 `List[<item_type>]`
   - `"object"`: `ALLOW_INLINE_COMPILATION`이 `True`이고 `"properties"`가 있으면 인라인 헬퍼 클래스 이름을 결정하고 `INLINE_OBJECTS`에 등록 후 이름 반환. 그렇지 않으면 `"Dict[str, Any]"`.
6. 매칭 없으면 `"Any"`.

인라인 클래스 이름 결정 규칙 (`ALLOW_INLINE_COMPILATION` 활성 시):
- `prop_name`이 `"ies"`로 끝나면: 마지막 3자를 `"y"`로 교체 후 대문자화 + `"Item"` (예: `"categories"` → `"CategoryItem"`)
- `"s"`로 끝나고 `"ss"`로 끝나지 않으면: 마지막 `"s"` 제거 후 대문자화 + `"Item"` (예: `"tabs"` → `"TabItem"`)
- 그 외: 프로퍼티 스키마의 첫 번째 내부 키를 대문자화 (예: 첫 키가 `"svgPath"` → `"SvgPath"`)

---

### `to_snake_case(name: str) -> str`

camelCase/PascalCase 문자열을 snake_case로 변환한다.

- 예외 처리: `name`이 `"v0_9"` 이면 그대로 반환 (버전 문자열 보호).
- 정규식 두 단계: `(.)([A-Z][a-z]+)` → `\1_\2`, `([a-z0-9])([A-Z])` → `\1_\2`, 결과를 `.lower()`.

---

### `compile_properties_to_pydantic(props: Dict[str, Any], required: List[str]) -> List[str]`

여러 JSON Schema 프로퍼티 정의를 받아 각각에 대한 Pydantic `Field` 선언 문자열 리스트를 반환한다.

각 프로퍼티 처리 흐름:
1. 프로퍼티 이름이 `"component"` 이면 건너뜀.
2. `map_json_type_to_python`으로 Python 타입 획득.
3. `description`이 있으면 `Field` 옵션에 추가 (줄바꿈을 공백으로 치환).
4. `"pattern"` 이 있으면 `Field` 옵션에 `pattern="<값>"` 추가.
5. `"default"` 가 있으면 `has_default = True`, 값이 `str`이면 따옴표 포함 문자열로, 아니면 그대로 `default=<값>`.
6. `"const"` 가 있으면 `has_default = True`, 동일하게 처리.
7. `to_snake_case`로 필드 이름을 변환하고, 원본과 다르면 `alias="<원본>"` 옵션 추가 (옵션 리스트의 맨 앞에 삽입).
8. `required` 목록에 있으면:
   - `"const"` 도 있으면: `Field(<const_값>, <alias/desc/pattern 옵션>)` (default 옵션 제거)
   - 그 외: `Field(..., <옵션들>)`
9. `required`에 없으면:
   - `has_default`이면: `Optional[<타입>] = Field(<default 값, 옵션>)` (default= 앞의 `, ` 제거)
   - 없으면: `Optional[<타입>] = Field(None, <옵션>)`

반환값은 `"    <snake_name>: <타입> = Field(...)"` 형태의 문자열 리스트 (들여쓰기 4칸 포함).

---

### `compile_component_to_pydantic(name: str, schema: Dict[str, Any], base_class: str = "ComponentCommon", common_data: Dict[str, Any] = None) -> str`

하나의 컴포넌트에 대한 Pydantic 클래스 소스 문자열을 생성한다.

- 클래스 선언: `class <name>Component(<base_class>):`
- `component` 필드: `Literal["<name>"] = "<name>"` (고정 판별자 필드)
- `schema`에 `"allOf"`가 있으면 각 파트를 순회하며 `properties`, `required`를 누적; `$ref`가 `common_types.json`을 참조하면 `common_data`에서 `$defs/<ref_name>`을 찾아 해당 프로퍼티도 포함.
- `"properties"` 직접 포함 시에는 그것을 사용.
- 상속된 공통 필드 `"id"`, `"accessibility"`, `"weight"` 는 필터링하여 중복 방지.
- `compile_properties_to_pydantic`에 위임하여 필드 생성.
- 반환: 줄바꿈으로 합친 클래스 코드 문자열 (끝에 `\n`).

---

### `compile_object_def(class_name: str, spec: Dict[str, Any], base_class: str = None) -> str`

표준 JSON Schema 객체 정의로부터 Pydantic 클래스 소스 문자열을 생성한다.

- `spec.get("additionalProperties")` 가 truthy이면 베이스 클래스를 `BaseModel`로 사용하고 `model_config = ConfigDict(populate_by_name=True)` 라인을 추가. 아니면 `StrictBaseModel` 사용.
- `base_class` 인자로 오버라이드 가능.
- `properties`가 없으면 `pass` 한 줄.
- 있으면 `compile_properties_to_pydantic`에 위임.

---

### `compile_union_def(class_name: str, spec: Dict[str, Any]) -> str`

JSON Schema `oneOf`/`anyOf`/`allOf` 리스트로부터 Python 타입 별칭 문자열을 생성한다.

- `union_items`가 없으면 `"<class_name> = Any"` 반환.
- 각 항목에 대해: `"allOf"`가 있으면 `allOf[0]`으로 unwrap.
- `map_json_type_to_python`으로 타입 매핑. 결과가 `"List[Any]"`이고 `items`가 있으면 item 타입을 재확인.
- 중복 제거 후 `"<class_name> = Union[...]"` 반환.

---

### `compile_function_to_pydantic(name: str, schema: Dict[str, Any]) -> tuple[str, str]`

하나의 카탈로그 함수를 두 개의 Pydantic 클래스 소스로 변환하여 `(소스코드 문자열, Api클래스명)` 튜플을 반환한다.

- `args` 프로퍼티가 있으면 `<Name>Args(StrictBaseModel)` 클래스 생성.
- `<Name>Api(FunctionApi)` 클래스 생성: `name = "<name>"`, `args = <Name>Args | None`, `return_type = "<returnType.const | 'boolean'>"`.

---

### `generate_schema_constants() -> str`

`schema/constants.py`의 내용 문자열을 반환한다. `SPEC_VERSION = "v0.9"` 한 줄만 포함.

---

### `generate_common_types(common_data: Dict[str, Any]) -> str`

`schema/common_types.py` 전체 소스를 생성한다.

생성 순서:
1. import 선언부 및 `ComponentReference`, `SingleReference`, `ListReference`, `StrictBaseModel` 인라인 정의 (하드코딩된 문자열).
2. `ComponentId = SingleReference` 타입 별칭.
3. `DataBinding`, `FunctionCall` — `compile_object_def` 사용.
4. `DynamicValue`, `DynamicString`, `DynamicNumber`, `DynamicBoolean`, `DynamicStringList` — `compile_union_def` 사용.
5. `TemplateChildList` — `ChildList.oneOf[1]`을 `StrictBaseModel, ListReference` 베이스로 생성; `ChildList = Union[List[ComponentId], TemplateChildList]`.
6. `AccessibilityAttributes`, `CheckRule` — `compile_object_def`.
7. `ActionEvent` — `Action.oneOf[0].properties.event`를 `compile_object_def`로 생성.
   `ActionEventWrapper` — `Action.oneOf[0]`의 deepcopy에서 `event` 프로퍼티를 `$ref` 형식으로 교체.
   `ActionFunctionCallWrapper` — `Action.oneOf[1]`.
   `Action = Union[ActionEventWrapper, ActionFunctionCallWrapper]`.
8. `ComponentCommon` — `compile_object_def`.

---

### `generate_basic_catalog_components(catalog_data: Dict[str, Any], common_data: Dict[str, Any] = None) -> tuple[str, List[str]]`

`basic_catalog/components.py` 전체 소스와 클래스 이름 목록을 반환한다.

- 전역 `ALLOW_INLINE_COMPILATION = True`로 설정 후 `INLINE_OBJECTS.clear()`.
- `CatalogComponentCommon(ComponentCommon)` 클래스를 `$defs.CatalogComponentCommon`로부터 생성 (없으면 `pass`).
- `catalog_data["components"]`를 순회하며 `compile_component_to_pydantic`으로 각 컴포넌트 코드 생성. 이 과정에서 `INLINE_OBJECTS`에 인라인 헬퍼 클래스가 누적됨.
- 발견된 인라인 객체들을 알파벳순으로 정렬 후 `compile_object_def`로 컴파일.
- 출력 순서: import → `CatalogComponentCommon` → 인라인 헬퍼 클래스들 → 컴포넌트 클래스들.
- `$defs.anyComponent.oneOf`에 `$ref`가 있으면 해당 이름의 클래스만 `AnyComponent Union`에 포함; 없으면 전체 컴포넌트 포함.
- `AnyComponent`는 Pydantic Discriminated Union으로 생성: `Annotated[Union[...], Field(..., discriminator="component")]` — O(1) 라우팅, 정확한 오류 보고, 타입 안전 역직렬화를 위함.
- 반환하는 이름 목록은 인라인 헬퍼 → `CatalogComponentCommon` → 컴포넌트 클래스들 → `AnyComponent` 순.
- 함수 종료 시 `ALLOW_INLINE_COMPILATION = False` 복원.

---

### `generate_basic_catalog_functions(catalog_data: Dict[str, Any]) -> tuple[str, List[str]]`

`basic_catalog/function_apis.py` 전체 소스와 API 클래스 이름 목록을 반환한다.

- `catalog_data["functions"]`를 순회하며 `compile_function_to_pydantic`으로 각 함수 코드 생성.
- `$defs.anyFunction.oneOf`가 있으면 해당 함수 이름만 반환 목록에 포함; 없으면 전체.

---

### `generate_basic_catalog_styles(catalog_data: Dict[str, Any]) -> str`

`basic_catalog/styles.py` 전체 소스를 반환한다.

- `$defs.theme` 스펙이 있으면 `Theme` 클래스를 `compile_object_def`로 생성; 없으면 `class Theme(BaseModel): pass`.

---

### `generate_catalog_functions() -> str`

`catalog/functions.py` 전체 소스를 반환한다.

- `FunctionApi` 베이스 클래스를 하드코딩된 문자열로 생성: `name: str = ""`, `args: Optional[Any] = None`, `return_type: str = "void"`.

---

### `generate_server_to_client(s2c_data: Dict[str, Any]) -> tuple[str, List[str]]`

`schema/server_to_client.py` 전체 소스와 메시지 클래스 이름 목록을 반환한다.

- `$defs`의 각 항목 (`*Message`)에 대해 두 클래스 생성:
  1. 페이로드 클래스 (`<Name>` — `Message` 제거): 메시지 스키마에서 첫 번째 `required` 키의 `properties`를 추출하여 필드 생성.
  2. 메시지 엔벨로프 클래스 (`<Name>Message`): `version: Literal[SPEC_VERSION] = SPEC_VERSION` + 페이로드 필드 (alias 포함).
- `A2uiMessage = Union[<모든 메시지 클래스>]`.
- `A2uiMessageListWrapper(StrictBaseModel)`: `messages: List[A2uiMessage]` 필드.

---

### `generate_client_capabilities(capabilities_data: Dict[str, Any]) -> str`

`schema/client_capabilities.py` 전체 소스를 반환한다.

- `$defs.FunctionDefinition` → `compile_object_def("FunctionDefinition", ...)`.
- `$defs.Catalog` → `compile_object_def("InlineCatalog", ...)` (이름 충돌 방지를 위해 rename).
- `V09Capabilities(StrictBaseModel)`: `properties.<SPEC_VERSION_DOT>.properties`에서 필드 생성.
- `A2uiClientCapabilities(StrictBaseModel)`: `v0_9: Optional[V09Capabilities] = Field(None, alias=SPEC_VERSION)`.
- 생성 후 `"List[Catalog]"` → `"List[InlineCatalog]"` 문자열 치환.

---

### `generate_client_to_server(c2s_data: Dict[str, Any]) -> str`

`schema/client_to_server.py` 전체 소스를 반환한다.

- `properties.action` → `A2uiClientAction` (`compile_object_def`).
- `properties.error.oneOf` 순회: title이 `"Validation Failed Error"` → `A2uiValidationError`, `"Generic Error"` → `A2uiGenericError`, 그 외 → 공백 제거한 title.
- `A2uiClientError = Union[<에러 클래스들>]`.
- `A2uiClientActionMessage`, `A2uiClientErrorMessage` (각각 `version` + 페이로드 필드).
- `A2uiClientMessage = Union[A2uiClientActionMessage, A2uiClientErrorMessage]`.
- `A2uiClientDataModel`: `version` + `surfaces: Dict[str, Dict[str, Any]]`.
- `A2uiClientMessageList = List[A2uiClientMessage]`.
- `A2uiClientMessageListWrapper`: `messages: A2uiClientMessageList`.

---

### `generate_schema_init(msg_names: List[str]) -> str`

`schema/__init__.py` 소스를 반환한다.

- `common_types`에서 핵심 타입들 re-export.
- `constants`에서 전체 re-export (`*`).
- `server_to_client`에서 각 메시지 이름과 페이로드 이름, `A2uiMessage`, `A2uiMessageListWrapper` re-export.
- `client_capabilities`에서 `A2uiClientCapabilities`, `V09Capabilities`, `InlineCatalog`, `FunctionDefinition` re-export.
- `client_to_server`에서 클라이언트 메시지 관련 전체 타입 re-export.

---

### `main() -> None`

스크립트 진입점. 다음 순서로 파일을 생성한다:

1. `SCHEMA_DIR`, `BASIC_CATALOG_DIR`, `CATALOG_DIR` 디렉터리 생성 (`exist_ok=True`).
2. `common_types.json` 로드 → `generate_common_types` → `schema/common_types.py` 쓰기.
3. `CONSTANTS_OUT_PATH` 미존재 시에만 → `generate_schema_constants` → `schema/constants.py` 쓰기.
4. `CATALOG_FUNCTIONS_OUT_PATH` 미존재 시에만 → `generate_catalog_functions` → `catalog/functions.py` 쓰기.
5. `catalog.json` 로드 → `generate_basic_catalog_components` → `basic_catalog/components.py` 쓰기.
6. (같은 `catalog_data`) → `generate_basic_catalog_functions` → `basic_catalog/function_apis.py` 쓰기.
7. (같은 `catalog_data`) → `generate_basic_catalog_styles` → `basic_catalog/styles.py` 쓰기.
8. `server_to_client.json` 로드 → `generate_server_to_client` → `schema/server_to_client.py` 쓰기.
9. `client_capabilities.json` 로드 → `generate_client_capabilities` → `schema/client_capabilities.py` 쓰기.
10. `client_to_server.json` 로드 → `generate_client_to_server` → `schema/client_to_server.py` 쓰기.
11. `generate_schema_init` → `schema/__init__.py` 쓰기.
12. 생성된 모든 파일에 대해 `uvx pyink <파일들>` 실행으로 자동 포매팅 (실패 시 경고만 출력하고 계속).

모든 파일 쓰기는 `FILE_HEADER` 접두사를 붙인다. `constants.py`와 `catalog/functions.py`는 idempotent 처리를 위해 이미 존재하면 재생성을 건너뛴다.

## 동작 흐름

```
[JSON 사양 파일들] → main() 로드
    → generate_common_types() → common_types.py
    → generate_schema_constants() → constants.py (없을 때만)
    → generate_catalog_functions() → catalog/functions.py (없을 때만)
    → generate_basic_catalog_components() → basic_catalog/components.py
        내부: ALLOW_INLINE_COMPILATION=True 동안 INLINE_OBJECTS 누적
    → generate_basic_catalog_functions() → basic_catalog/function_apis.py
    → generate_basic_catalog_styles() → basic_catalog/styles.py
    → generate_server_to_client() → schema/server_to_client.py
    → generate_client_capabilities() → schema/client_capabilities.py
    → generate_client_to_server() → schema/client_to_server.py
    → generate_schema_init() → schema/__init__.py
    → uvx pyink 포매팅
```

모든 코드 생성 함수는 문자열 리스트(`output`)를 구성하고 `"\n".join(output)`으로 합쳐 반환하는 패턴을 공유한다. 타입 변환의 핵심은 `map_json_type_to_python` → `compile_properties_to_pydantic` → 각 `compile_*` 함수의 계층 구조다.
