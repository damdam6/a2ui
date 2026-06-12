# agent_sdks/python/a2ui_core/src/a2ui/core/validating/catalog_validator.py

## 개요

카탈로그 스키마에 대한 컴포넌트·함수·테마 유효성 검증 로직을 제공한다. 추상 기반 클래스 `CatalogValidator`와 두 가지 구체 구현(`ModelCatalogValidator`는 Pydantic 모델 기반, `JsonCatalogValidator`는 JSON Schema Draft 2020-12 기반)으로 구성된다. 카탈로그 종류(`ModelCatalog` 또는 `JsonCatalog`)에 따라 팩토리 메서드 `from_catalog`가 적절한 구현체를 반환한다.

## 의존성

### 외부 패키지
- `typing`: `Any`, `Dict`, `List`, `Optional`, `Set`, `Tuple`
- `jsonschema`: `Draft202012Validator`
- `referencing`: `Registry`, `Resource`
- `referencing.jsonschema`: `DRAFT202012`

### 저장소 내부 모듈
- [`../catalog/__init__.py`](../catalog/__init__.py.md) — `CatalogApi`, `JsonCatalog`, `ModelCatalog`
- [`../schema/constants.py`](../schema/constants.py.md) — `CATALOG_COMPONENTS_KEY`, `SPEC_BASE_URL`

## Exports

| 이름 | 종류 |
|---|---|
| `CatalogValidator` | 클래스 (추상 기반) |
| `ModelCatalogValidator` | 클래스 |
| `JsonCatalogValidator` | 클래스 |

## 상세 명세

### 모듈 수준 상수

- `JSON_SCHEMA_DRAFT_2020_12`: `"https://json-schema.org/draft/2020-12/schema"` — JSON Schema Draft 버전 URI
- `COMMON_TYPES_SCHEMA_FILE`: `"common_types.json"` — 공통 타입 스키마 파일명
- `CATALOG_SCHEMA_FILE`: `"catalog.json"` — 카탈로그 스키마 파일명

### `_schema_url(spec_version: str, file_name: str) -> str` (모듈 내부 헬퍼)

스펙 버전과 파일명으로부터 정규 스키마 URL을 생성한다. `spec_version`이 `"v"` 접두사로 시작하지 않으면 `f"v{spec_version}"`으로 추가한다. 버전 문자열의 `.`을 `_`로 모두 치환하여 `f"{SPEC_BASE_URL}/{ver}/{file_name}"` 형태의 URL을 반환한다. 예: `spec_version="0.9"`, `file_name="catalog.json"` → `"https://a2ui.org/specification/v0_9/catalog.json"`.

---

### `CatalogValidator` (추상 기반 클래스)

모든 카탈로그 검증기의 공통 인터페이스를 정의한다.

**`__init__(self, catalog: CatalogApi) -> None`**

`self.catalog`에 `CatalogApi` 인스턴스를 저장한다.

**`validate_components(self, comp_payload: List[Dict[str, Any]]) -> None`**

컴포넌트 페이로드 목록을 카탈로그 스키마에 대해 검증한다. 서브클래스에서 반드시 구현해야 하며, 기본 구현은 `NotImplementedError("Subclasses must implement validate_components()")`를 발생시킨다.

**`validate_function(self, func_name: str, args: Dict[str, Any]) -> None`**

함수 인수가 카탈로그의 해당 함수 스키마에 부합하는지 검증한다. 기본 구현은 `NotImplementedError("Subclasses must implement validate_function()")`를 발생시킨다.

**`validate_theme(self, theme_payload: Dict[str, Any]) -> None`**

테마 속성이 카탈로그의 테마 스키마에 부합하는지 검증한다. 기본 구현은 `NotImplementedError("Subclasses must implement validate_theme()")`를 발생시킨다.

**`extract_ref_fields(self) -> Dict[str, Tuple[Set[str], Set[str]]]`**

카탈로그의 토폴로지 참조 포인터 맵을 반환한다. 내부적으로 `self.catalog.extract_ref_fields()`에 전적으로 위임하며 별도 로직이 없다.

**`from_catalog(cls, catalog: CatalogApi) -> "CatalogValidator"` (classmethod)**

`catalog`의 타입을 `isinstance` 검사로 판별하여 적절한 검증기 인스턴스를 생성해 반환하는 팩토리 메서드다. `isinstance(catalog, ModelCatalog)`이면 `ModelCatalogValidator(catalog)`를 반환하고, `isinstance(catalog, JsonCatalog)`이면 `JsonCatalogValidator(catalog)`를 반환한다. 두 타입 모두 해당하지 않으면 `ValueError(f"No CatalogValidator implementation found for catalog type '{type(catalog).__name__}'")`를 발생시킨다.

---

### `ModelCatalogValidator(CatalogValidator)`

Pydantic 모델 기반 카탈로그를 검증하는 구현체.

**`__init__(self, catalog: ModelCatalog) -> None`**

`super().__init__(catalog)`을 호출하고, `self.catalog`를 `ModelCatalog` 타입으로 재선언(타입 힌트 재지정)한다.

**`_check_nested_functions(self, val: Any) -> None` (비공개 헬퍼)**

`val` 내부를 재귀적으로 탐색하며 함수 호출 패턴 `{"call": ..., "args": ...}`를 발견하면 검증한다.
- `val`이 리스트이면 각 항목에 대해 재귀 호출한다.
- `val`이 딕셔너리이면, `"call"` 키와 `"args"` 키가 동시에 있으면 `func_name = val["call"]`으로 `validate_function(func_name, val["args"])`를 시도하고, 실패하면 `ValueError(f"Invalid function call '{func_name}': {e}")`로 재포장한다. 그 후 딕셔너리의 모든 값에 대해 재귀 호출한다.

**`_validate_component(self, comp_type: str, comp_payload: Dict[str, Any]) -> None` (비공개)**

단일 컴포넌트 페이로드를 Pydantic으로 검증한다.
1. `catalog.get_component_class(comp_type)`로 컴포넌트 클래스를 조회한다. `None`이면 `ValueError(f"Unknown component type: {comp_type}")`를 발생시킨다.
2. `comp_class`에 `model_json_schema` 속성이 있으면 호출하여 JSON 스키마 dict를 얻는다. 없으면 빈 dict를 사용한다.
3. 스키마의 `"unevaluatedProperties"`가 `False`이면: `comp_class`에 `model_fields`가 있으면 그 키 집합을 `defined`로 사용하고, `comp_payload`에서 `defined`에 없고 `"component"`도 아닌 키를 `extra` 목록으로 수집한다. `extra`가 비어 있지 않으면 `ValueError(f"Extra inputs are not permitted: {extra}")`를 발생시킨다.
4. `comp_class.model_validate(comp_payload)`로 Pydantic 검증을 수행한다.
5. `_check_nested_functions(comp_payload)`로 중첩 함수 호출을 검증한다.

**`validate_components(self, comp_payload: List[Dict[str, Any]]) -> None`**

`comp_payload`를 순회하며 항목이 딕셔너리이고 `"component"` 키가 있는 경우에만 `_validate_component(comp["component"], comp)`를 호출한다.

**`validate_theme(self, theme_payload: Dict[str, Any]) -> None`**

`self.catalog.theme`가 truthy이면 `self.catalog.theme.model_validate(theme_payload)`를 호출한다. `catalog.theme`가 `None`이거나 falsy이면 아무것도 하지 않는다.

**`validate_function(self, func_name: str, args: Dict[str, Any]) -> None`**

`catalog.get_function_class(func_name)`으로 함수 클래스를 가져온다. `None`이면 `ValueError(f"Unknown function: {func_name}")`를 발생시킨다. 함수 클래스에 `model_fields` 속성이 있고 `"call"`이 그 키에 포함되어 있으면 `{"call": func_name, "args": args}` 페이로드를 만들어 `func_class.model_validate(payload)`를 호출한다. 그렇지 않으면 `func_class.model_validate(args)`를 직접 호출한다.

---

### `JsonCatalogValidator(CatalogValidator)`

JSON Schema Draft 2020-12 기반 카탈로그를 검증하는 구현체.

**`__init__(self, catalog: JsonCatalog) -> None`**

`super().__init__(catalog)`을 호출하고 `self.catalog`를 `JsonCatalog`로 재선언한다. 빈 딕셔너리 `self._validators: Dict[str, Draft202012Validator]`를 초기화하고, `self._registry = self._build_registry()`를 호출한다.

**`_build_registry(self) -> Registry` (비공개)**

jsonschema `Registry`를 구성한다. `resources` 리스트를 만들어 다음을 추가한다:
1. `(CATALOG_SCHEMA_FILE, Resource.from_contents(catalog.catalog_schema, DRAFT202012))` — 단순 파일명 키
2. `(_schema_url(catalog.spec_version, CATALOG_SCHEMA_FILE), Resource.from_contents(catalog.catalog_schema, DRAFT202012))` — 버전화된 URL 키
3. `catalog.common_types_schema`가 truthy이면, 마찬가지로 `COMMON_TYPES_SCHEMA_FILE` 단순 키와 버전화된 URL 키 두 쌍을 추가한다.

최종적으로 `Registry().with_resources(resources)`를 반환한다.

**`_get_validator(self, key: str, ref_path: str) -> Draft202012Validator` (비공개)**

`key`가 `self._validators` 캐시에 없으면 `{"$schema": JSON_SCHEMA_DRAFT_2020_12, "$ref": ref_path}` 스키마와 `registry=self._registry`로 `Draft202012Validator`를 생성하여 캐시에 저장한다. 생성 중 예외가 발생하면 `ValueError(str(e))`로 변환한다. 캐시에 있으면 그대로 반환한다.

**`validate_component_properties(self, comp_type: str, properties: Dict[str, Any]) -> None`**

컴포넌트 타입의 스키마를 직접 조회하여 `properties` 딕셔너리를 검증한다. `catalog.get_component_schema(comp_type)`가 `None`이면 `ValueError(f"Unknown component type: {comp_type}")`를 발생시킨다. `_get_validator("comp:{comp_type}", "catalog.json#/components/{comp_type}")`로 검증기를 얻어 `list(validator.iter_errors(properties))`를 구한다. 오류가 있으면 모든 오류의 `message`를 `"\n"`으로 이어 붙여 `ValueError`를 발생시킨다.

**`_validate_component(self, comp_type: str, comp_payload: Dict[str, Any]) -> None` (비공개)**

JSON Schema 규칙으로 단일 컴포넌트 페이로드를 검증한다.
1. 내부 헬퍼 `defines_property(schema: Any, prop_name: str) -> bool`를 정의한다. 이 함수는 `schema` dict의 `"properties"` 키에 `prop_name`이 있으면 `True`를 반환한다. `"allOf"`, `"oneOf"`, `"anyOf"` 키 하위의 각 서브스키마에 대해 재귀적으로 `defines_property`를 호출한다. `"$ref"` 값에 `"ComponentCommon"`이 포함되어 있고 `prop_name == "id"`이면 `True`를 반환한다. 위 조건이 모두 해당하지 않으면 `False`를 반환한다.
2. `strip_keys = []`를 초기화하고, `defines_property(comp_schema, "id")`가 `False`이면 `"id"`를 추가하고, `defines_property(comp_schema, "component")`가 `False`이면 `"component"`를 추가한다.
3. `properties = {k: v for k, v in comp_payload.items() if k not in strip_keys}`로 제거 대상 키를 걸러낸다.
4. `validate_component_properties(comp_type, properties)`를 호출하고 `_check_nested_functions(comp_payload)`를 호출한다. 예외 발생 시 `ValueError(str(e))`로 재포장한다.

**`validate_components(self, comp_payload: List[Dict[str, Any]]) -> None`**

`comp_payload`를 순회하며 딕셔너리이고 `"component"` 키가 있는 항목에 대해 `_validate_component(comp["component"], comp)`를 호출한다.

**`validate_theme(self, theme_payload: Dict[str, Any]) -> None`**

`catalog.catalog_schema.get("theme")`가 falsy이면 즉시 반환한다. `catalog.catalog_schema`에 `"$defs"` 키가 있고 그 하위에 `"theme"` 키가 있으면 `ref_path = "catalog.json#/$defs/theme"`를 사용하고, 그렇지 않으면 `ref_path = "catalog.json#/theme"`를 사용한다. `_get_validator("theme:schema", ref_path)`로 검증기를 얻어 `list(validator.iter_errors(theme_payload))`를 구한다. 오류가 있으면 첫 번째 오류의 `message`를 `ValueError`로 발생시킨다.

**`validate_function(self, func_name: str, args: Dict[str, Any]) -> None`**

`catalog.get_function_schema(func_name)`으로 함수 스키마를 조회한다. `None`이면 `ValueError(f"Unknown function: {func_name}")`를 발생시킨다. `_get_validator(f"func:{func_name}", "catalog.json#/functions/{func_name}")`로 검증기를 얻고, `{"call": func_name, "args": args}` 페이로드를 `list(validator.iter_errors(payload))`로 검증한다. 오류가 있으면 첫 번째 오류의 `message`를 `ValueError`로 발생시킨다.

**`_check_nested_functions(self, val: Any) -> None` (비공개)**

`ModelCatalogValidator._check_nested_functions`와 동일한 로직. 재귀적으로 `val`을 탐색하여 `{"call": ..., "args": ...}` 패턴을 발견하면 `self.validate_function(func_name, val["args"])`를 호출하고, 예외 발생 시 `ValueError(f"Invalid function call '{func_name}': {e}")`로 재포장한다.

## 동작 흐름

외부 코드는 일반적으로 `CatalogValidator.from_catalog(catalog)` 팩토리 메서드로 검증기를 생성한다. 이후 `validate_components`, `validate_function`, `validate_theme`를 독립적으로 호출할 수 있다. `JsonCatalogValidator`는 초기화 시 `_build_registry`를 호출하여 jsonschema 레지스트리를 준비하고, 이후 각 컴포넌트/함수/테마 타입별 검증기를 지연 생성(lazy cache)하여 `_get_validator`의 `_validators` 딕셔너리에 저장한다. 두 구현 모두 중첩 함수 호출(`{"call": ..., "args": ...}`)을 재귀 탐색하는 `_check_nested_functions` 헬퍼를 보유하며, 이를 통해 컴포넌트 속성 값에 포함된 함수 참조도 검증된다.
