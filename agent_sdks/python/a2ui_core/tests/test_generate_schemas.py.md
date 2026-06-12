# agent_sdks/python/a2ui_core/tests/test_generate_schemas.py

## 개요

`scripts/generate_schemas.py` 코드 생성 스크립트의 개별 함수들을 단위 테스트하는 파일이다. JSON Schema 정의를 Pydantic 모델 소스 코드로 변환하는 각 함수가 올바른 Python 코드 문자열을 생성하는지 검증한다. 실제 JSON 스키마 파일을 읽지 않고 인라인 mock 데이터를 직접 구성하여 독립적으로 실행되며, 모듈 로드 시 동적으로 `../scripts` 경로를 `sys.path`에 삽입해 `generate_schemas`를 가져온다.

## 의존성

### 외부 패키지
- `sys`, `os` — `../scripts` 경로를 `sys.path`에 동적으로 추가하기 위해 사용
- `pytest` — 테스트 프레임워크

### 저장소 내부 모듈
- [`agent_sdks/python/a2ui_core/scripts/generate_schemas.py`](../scripts/generate_schemas.py.md) — 테스트 대상 스크립트. `sys.path` 동적 주입 후 `import generate_schemas`로 로드됨.

## Exports

테스트 파일이므로 공개 export 없음.

## 모듈 초기화

파일 최상단에서 `__file__` 기준으로 `../scripts` 절대 경로를 계산한다. 해당 경로가 `sys.path`에 없으면 인덱스 0에 삽입한 뒤 `generate_schemas`를 import한다.

```
SCRIPT_DIR   = os.path.dirname(os.path.abspath(__file__))
SCRIPTS_PATH = os.path.abspath(os.path.join(SCRIPT_DIR, "../scripts"))
```

## 동작 흐름 (테스트 케이스 상세)

### `test_map_json_type_to_python`

**검증 대상:** `generate_schemas.map_json_type_to_python(prop_name: str, prop_schema: dict) -> str`

JSON Schema 속성 딕셔너리를 Python 타입 문자열로 변환하는 함수의 모든 분기를 검사한다. 픽스처/모킹 없음.

| 입력 케이스 | 기대 출력 |
|---|---|
| `{"$ref": "common_types.json#/$defs/ComponentId"}` | `"ComponentId"` |
| `{"$ref": "common_types.json#/$defs/DynamicString"}` | `"DynamicString"` |
| `{"$ref": "#/$defs/CatalogComponentCommon"}` | `"CatalogComponentCommon"` |
| `{"$ref": "other.json#/$defs/Unknown"}` (지원 안 되는 외부 ref) | `"Any"` |
| `{"oneOf": [{"type": "string"}, {"type": "integer"}]}` | `"Union[str, int]"` |
| `{"anyOf": [{"type": "boolean"}]}` (단일 항목) | `"bool"` (Union 래핑 없음) |
| `{"allOf": [{"$ref": "common_types.json#/$defs/DynamicString"}, {"if": ...}]}` | `"DynamicString"` (첫 번째 `$ref`에서 타입 추출) |
| `{"type": "string"}` | `"str"` |
| `{"type": "string", "enum": ["small", "large"]}` | `'Literal["small", "large"]'` |
| `{"type": "number"}` | `"float"` |
| `{"type": "integer"}` | `"int"` |
| `{"type": "boolean"}` | `"bool"` |
| `{"type": "array", "items": {"type": "string"}}` | `"List[str]"` |
| `{"type": "object"}` | `"Dict[str, Any]"` |
| `{}` (알 수 없는 스키마) | `"Any"` |
| `{"const": "SUCCESS"}` | `"Literal['SUCCESS']"` |
| `{"const": 404}` | `"Literal[404]"` |

---

### `test_compile_properties_to_pydantic`

**검증 대상:** `generate_schemas.compile_properties_to_pydantic(props: dict, required: list) -> list[str]`

속성 딕셔너리와 required 목록을 받아 Pydantic 필드 선언 문자열 리스트를 반환하는 함수를 검사한다. 픽스처/모킹 없음.

| 케이스 | 기대 출력 라인 |
|---|---|
| `title`이 required 목록에 있고 description 있음 | `'    title: str = Field(..., description="Simple title")'` |
| `title`이 optional (required 비어있음) | `"    title: Optional[str] = Field(None)"` |
| `num`에 `default: 42` | `"    num: Optional[int] = Field(default=42)"` |
| `text`에 `default: "hello"` | `'    text: Optional[str] = Field(default="hello")'` |
| 속성명이 `"component"` | 결과 리스트에 포함되지 않음 (skip) |

---

### `test_compile_component_to_pydantic`

**검증 대상:** `generate_schemas.compile_component_to_pydantic(name: str, schema: dict) -> str`

컴포넌트 스키마로부터 `ComponentCommon`을 상속하는 Pydantic 클래스 코드를 생성하는 함수를 검사한다. 픽스처/모킹 없음.

mock 입력: `properties`에 `component("MyComp" const)`, `id`, `text`, `accessibility` 포함. `required`에 `component`와 `text`.

검증 항목:
- `"class MyCompComponent(ComponentCommon):"` 포함
- `'    component: Literal["MyComp"] = "MyComp"'` 포함
- `"    text: str = Field(...)"` 포함
- `"id: "` 미포함 (상속 필드 skip)
- `"accessibility: "` 미포함 (상속 필드 skip)

---

### `test_compile_object_def`

**검증 대상:** `generate_schemas.compile_object_def(name: str, spec: dict) -> str`

일반 객체 정의를 `StrictBaseModel` 또는 `BaseModel`을 상속하는 클래스로 변환하는 함수를 검사한다. 픽스처/모킹 없음.

| 케이스 | 기대 동작 |
|---|---|
| 기본 spec (`required: ["x"]`) | `class Point(StrictBaseModel):`, `x: float = Field(...)` 포함 |
| `additionalProperties: True` | `class Point(BaseModel):` (유연한 모델) |
| 빈 spec `{}` | `class Empty(StrictBaseModel):`, `    pass` 포함 |

---

### `test_compile_union_def`

**검증 대상:** `generate_schemas.compile_union_def(name: str, spec: dict) -> str`

`oneOf` 스키마를 Python `Union` 타입 별칭으로 변환하는 함수를 검사한다. 픽스처/모킹 없음.

입력: `{"oneOf": [{"type": "string"}, {"$ref": "common_types.json#/$defs/DataBinding"}]}`
기대 출력: `"StringOrBinding = Union[str, DataBinding]\n"`

---

### `test_compile_function_to_pydantic`

**검증 대상:** `generate_schemas.compile_function_to_pydantic(name: str, schema: dict) -> tuple[str, str]`

함수 스키마로부터 `FunctionApi`를 상속하는 API 클래스와 args 클래스를 생성하는 함수를 검사한다. 반환값은 `(생성된_코드, 클래스명)` 튜플. 픽스처/모킹 없음.

**케이스 A — args 있는 함수 `"add"`:**
- `class_name == "AddApi"`
- `class AddArgs(StrictBaseModel):` 생성
- `x: int = Field(...)` 포함
- `class AddApi(FunctionApi):` 생성
- `name = "add"`, `args = AddArgs`, `return_type = "boolean"` 포함

**케이스 B — args 없는 함수 `"random"`:**
- `class_name == "RandomApi"`
- `class RandomApi(FunctionApi):` 생성
- `name = "random"`, `args = None`, `return_type = "number"` 포함

---

### `test_generate_common_types`

**검증 대상:** `generate_schemas.generate_common_types(common_data: dict) -> str`

`common_types.json` 스키마 데이터에서 공통 타입 모듈 코드를 생성하는 함수를 mock 데이터로 검사한다. 픽스처/모킹 없음.

mock 데이터의 `$defs` 키: `DataBinding`, `FunctionCall`, `DynamicValue`, `DynamicString`, `DynamicNumber`, `DynamicBoolean`, `DynamicStringList`, `ChildList`, `AccessibilityAttributes`, `CheckRule`, `Action`, `ComponentCommon`.

검증 항목:
- `"class StrictBaseModel(BaseModel):"` 포함
- `"ComponentId = SingleReference"` 포함
- `"class DataBinding(StrictBaseModel):"` 포함
- `"class FunctionCall(StrictBaseModel):"` 포함
- `"DynamicValue = Union[str]"` 포함
- `"ChildList = Union[List[ComponentId], TemplateChildList]"` 포함 (특수 처리)

---

### `test_generate_basic_catalog_components`

**검증 대상:** `generate_schemas.generate_basic_catalog_components(catalog_data: dict) -> tuple[str, list[str]]`

세 가지 시나리오로 카탈로그 컴포넌트 코드 생성 함수를 검사한다. 픽스처/모킹 없음.

**시나리오 A — `$defs/anyComponent` 없는 경우 (전체 컴포넌트 폴백):**
- 입력: `components.Text` 하나만 존재
- `names == ["CatalogComponentCommon", "TextComponent", "AnyComponent"]`
- `CatalogComponentCommon`, `TextComponent` 클래스 생성
- `"AnyComponent = Annotated["`, `"TextComponent,"` 포함

**시나리오 B — `$defs/anyComponent/oneOf` 있는 경우 (교집합):**
- 입력: `components`에 `Text`, `PrivateHelper`; `$defs.anyComponent.oneOf`에 `Text`, `NonExistent`
- `names_defs == ["CatalogComponentCommon", "TextComponent", "AnyComponent"]` (교집합)
- 코드에는 `TextComponent`와 `PrivateHelperComponent` 클래스가 모두 생성됨
- `AnyComponent` union에는 `TextComponent`만 포함; `PrivateHelperComponent`, `NonExistentComponent`는 미포함

**시나리오 C — Icon 컴포넌트 내 SvgPath 감지:**
- 입력: `Icon.name`이 `oneOf`로 string enum(`["add", "close"]`)과 `svgPath` 객체를 가짐
- `"SvgPath" in names_svg`
- `"class SvgPath(StrictBaseModel):"` 생성
- `'    svg_path: str = Field(..., alias="svgPath")'` 포함
- `'Union[Literal["add", "close"], SvgPath]'` 타입 표현식 포함

---

### `test_generate_basic_catalog_functions`

**검증 대상:** `generate_schemas.generate_basic_catalog_functions(catalog_data: dict) -> tuple[str, list[str]]`

픽스처/모킹 없음.

**시나리오 A — `$defs/anyFunction` 없는 경우 (전체 함수 폴백):**
- `names == ["ToastApi"]`
- `"class ToastApi(FunctionApi):"` 포함

**시나리오 B — `$defs/anyFunction/oneOf` 있는 경우 (교집합):**
- 입력: `functions`에 `toast`, `privateFunc`; `$defs.anyFunction.oneOf`에 `toast`, `nonExistentFunc`
- `names_defs == ["ToastApi"]`
- 코드에는 `ToastApi`와 `PrivateFuncApi` 클래스가 모두 생성됨

---

### `test_generate_basic_catalog_styles`

**검증 대상:** `generate_schemas.generate_basic_catalog_styles(catalog_data: dict) -> str`

`$defs.theme` 스키마에서 `Theme` 클래스 코드를 생성하는 함수를 검사한다. 픽스처/모킹 없음.

mock 입력: `$defs.theme`에 `additionalProperties: True`와 camelCase 속성 `primaryColor`.

검증 항목:
- `"class Theme(BaseModel):"` 포함 (`additionalProperties: True`이므로 `BaseModel` 사용)
- `'primary_color: Optional[str] = Field(None, alias="primaryColor", description="Test color.")'` 포함

---

### `test_generate_server_to_client`

**검증 대상:** `generate_schemas.generate_server_to_client(s2c_data: dict) -> tuple[str, list[str]]`

server-to-client 메시지 스키마에서 Pydantic 클래스를 생성하는 함수를 검사한다. 픽스처/모킹 없음.

mock 입력: `$defs.CreateSurfaceMessage`에 required 필드 `createSurface.surfaceId` 포함.

검증 항목:
- `names == ["CreateSurfaceMessage"]`
- 내부 속성 `createSurface`도 별도 클래스 `class CreateSurface(StrictBaseModel):` 생성
- 외부 wrapper `class CreateSurfaceMessage(StrictBaseModel):` 생성

---

### `test_generate_schema_init`

**검증 대상:** `generate_schemas.generate_schema_init(names: list[str]) -> str`

스키마 패키지의 `__init__.py` 내용을 생성하는 함수를 검사한다. 픽스처/모킹 없음.

입력: `["CreateSurfaceMessage"]`

검증 항목:
- `"from .common_types import ("` 포함
- `"from .constants import *"` 포함
- `"    CreateSurfaceMessage,"` 포함
- `"    CreateSurface,"` 포함 (Message suffix 제거 후 내부 타입도 export)

---

### `test_generate_client_capabilities`

**검증 대상:** `generate_schemas.generate_client_capabilities(capabilities_data: dict) -> str`

클라이언트 capabilities 스키마에서 Pydantic 클래스 코드를 생성하는 함수를 검사한다. 픽스처/모킹 없음.

mock 입력: 최상위에 `"v0.9"` 버전 키와 `$defs.FunctionDefinition` 포함.

검증 항목:
- `"class FunctionDefinition(StrictBaseModel):"` 포함
- `"class V09Capabilities(StrictBaseModel):"` 포함 (`"v0.9"` → `"V09"` 변환)
- `"class A2uiClientCapabilities(StrictBaseModel):"` 포함
- `"v0_9: Optional[V09Capabilities] = Field(None, alias=SPEC_VERSION)"` 포함

---

### `test_const_keyword_mapping`

`map_json_type_to_python`과 `compile_properties_to_pydantic` 두 함수에서 `const` 키워드 처리를 추가 검증한다. 픽스처/모킹 없음.

- `{"const": "SUCCESS"}` → `"Literal['SUCCESS']"`
- `{"const": 404}` → `"Literal[404]"`
- required const 필드에서 `compile_properties_to_pydantic`은 `"    code: Literal['FAIL'] = Field(\"FAIL\")"` 형태를 생성

---

### `test_file_header_preamble`

**검증 대상:** `generate_schemas.FILE_HEADER` 상수 (문자열)

파일 헤더 문자열에 `"Copyright 2026 Google LLC"`와 `"Auto-generated. Do not edit manually."` 양쪽이 모두 포함되어 있는지 확인한다. 픽스처/모킹 없음.

---

### `test_generate_catalog_functions`

**검증 대상:** `generate_schemas.generate_catalog_functions() -> str`

인자 없이 호출하여 `FunctionApi` 기반 클래스 정의 코드를 생성하는 함수를 검사한다. 픽스처/모킹 없음.

검증 항목:
- `"class FunctionApi:"` 포함
- `'name: str = ""'` 포함
- `"args: Optional[Any] = None"` 포함
- `'return_type: str = "void"'` 포함
