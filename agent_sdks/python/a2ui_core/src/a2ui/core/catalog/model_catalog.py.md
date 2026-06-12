# agent_sdks/python/a2ui_core/src/a2ui/core/catalog/model_catalog.py

## 개요

Pydantic 모델 클래스들로 컴파일된 카탈로그의 구체 구현체인 `ModelCatalog`를 제공한다. `CatalogImplementation`을 상속해 컴포넌트 클래스 조회, 함수 스키마·구현체 조회, 함수 호출, 참조 필드 추출 기능을 모두 구현한다. 생성자에서 다양한 형태의 함수 입력(Pydantic 모델, `FunctionImplementation` 인스턴스, `FunctionApi` 클래스/인스턴스 등)을 정규화하고 내부 `invoker`를 구성한다.

## 의존성

### 외부 패키지
- `typing` — `Any`, `Callable`, `Dict`, `List`, `Optional`, `Set`, `Tuple`, `Type`, `Union`, `get_args`, `get_origin`
- `pydantic` — `BaseModel`

### 저장소 내부 모듈
- [`./catalog.py`](catalog.py.md) — `CatalogImplementation`
- [`./functions.py`](functions.py.md) — `FunctionImplementation` (직접), `FunctionApi` (지연 import)
- [`../schema/common_types.py`](../schema/common_types.py.md) — `ComponentReference`, `SingleReference`, `ListReference`

## Exports

| 이름 | 종류 |
|---|---|
| `ModelCatalog` | 클래스 |

## 상세 명세

### `ModelCatalog(CatalogImplementation)`

#### `__init__(self, spec_version: str, catalog_id: str, components: Dict[str, Type[BaseModel]], functions: Optional[Dict[str, Any]] = None, theme: Optional[Type[BaseModel]] = None)`

**초기화 단계:**

1. 부모 `CatalogImplementation.__init__(spec_version, catalog_id)` 호출.
2. `self.components`, `self.theme` 저장.
3. `self.functions: Dict[str, FunctionImplementation] = {}` 초기화.
4. `functions`가 주어진 경우, 딕셔너리가 아니면 리스트로 간주해 `fn.name`(또는 `fn.__name__`)을 키로 딕셔너리 변환.
5. 각 `(name, fn)` 쌍에 대해 fn의 종류를 판별해 정규화:
   - **`fn`이 `BaseModel` 서브클래스(type)**: `CoercedFunctionImplementation` 로컬 클래스로 래핑. `execute()`는 `None` 반환. `name`과 소문자 시작 camelCase 버전 두 가지 키로 등록.
   - **`fn`에 `execute` 메서드가 있음**: 그대로 사용. 두 가지 케이스로 등록.
   - **`fn`이 `FunctionApi` 서브클래스(type)**: 인스턴스화해 `api_inst.name`(또는 `name`)을 키로 등록. 대문자 시작 버전도 추가 등록.
   - **`fn`이 `FunctionApi` 인스턴스**: `fn.name`(또는 `name`) 키로 등록. 대문자 시작 버전도 추가 등록.
   - **`fn`에 `schema` 속성이 있음**: 그대로 사용. 두 가지 케이스로 등록.
6. `dynamic_invoker(name, args, context=None, abort_signal=None)` 내부 함수 정의:
   - `self.functions.get(name)` 조회 → 없으면 첫 글자 대문자 → 없으면 첫 글자 소문자 순으로 fallback.
   - `fn`이 존재하고 `execute` 속성이 있으면: `fn.schema`가 Pydantic 모델이면 인수 검증 수행. `fn.schema.model_fields`에 `"call"` 키가 있으면 `{"call": name, "args": args}` 형태로 검증, 없으면 `args`만 검증. 검증 실패 시 `ValueError` 발생.
   - `fn.execute(args, context, abort_signal)` 호출 후 반환.
7. `self.invoker = dynamic_invoker`.

---

#### `get_component_class(self, comp_type: str) -> Optional[Type[BaseModel]]`
`self.components.get(comp_type)` 반환.

#### `get_function_class(self, func_name: str) -> Optional[Type[BaseModel]]`
1. `func_name`이 비어 있으면 `None`.
2. 대문자 시작 → 원래 이름 → 소문자 시작 순으로 `self.functions` 조회.
3. 찾은 `fn`에서: `fn.schema` 속성이 있으면 반환 / `fn`이 `BaseModel` 서브클래스(type)이면 `fn` 반환 / `fn`이 `FunctionApi` 서브클래스(type)이면 인스턴스화 후 `.schema` 반환.

#### `get_component_schema(self, comp_type: str) -> Optional[Dict[str, Any]]`
`get_component_class(comp_type)`로 클래스를 얻은 뒤 `model_json_schema()` 호출.

#### `get_function_schema(self, func_name: str) -> Optional[Dict[str, Any]]`
`get_function_class(func_name)`으로 클래스를 얻은 뒤 `model_json_schema()` 호출.

#### `get_theme_schema(self) -> Optional[Dict[str, Any]]`
`self.theme`이 있고 `model_json_schema` 메서드가 있으면 호출해 반환.

#### `get_function_implementation(self, func_name: str) -> Optional[FunctionImplementation]`
`self.functions.get(func_name)` 직접 반환.

#### `invoke_function(self, name: str, args: Dict[str, Any], context: Any = None, abort_signal: Optional[Any] = None) -> Any`
`self.invoker(name, args, context, abort_signal)` 위임 호출.

#### `extract_ref_fields(self) -> Dict[str, Tuple[Set[str], Set[str]]]`

Pydantic 모델의 `model_fields`를 검사해 참조 필드 맵을 구성한다.

**내부 헬퍼 `_is_ref_type(typ: Any) -> Tuple[bool, bool]`:**
- `typ`이 `SingleReference` 서브클래스이면 `(True, False)`.
- `typ`이 `ListReference` 서브클래스이면 `(False, True)`.
- `get_origin(typ)`이 `list`/`List`이면: 첫 번째 type 인수 `elem`을 검사. `ComponentReference` 서브클래스이면 `(False, True)`. `BaseModel` 서브클래스이면 그 모델의 필드들을 재귀 검사해 하나라도 참조이면 `(False, True)`.
- `get_origin(typ) == Union`이면: 각 인수에 대해 재귀 검사, 하나라도 단일/리스트 참조가 있으면 합산.
- 그 외 `(False, False)`.

**메인 루프**: `self.components` 순회. 각 컴포넌트의 `model_fields`에서 `"id"`, `"component"` 필드는 건너뜀. 나머지 필드에 `_is_ref_type`을 적용해 집합 구성. 비어 있지 않으면 `ref_map`에 추가.

## 동작 흐름

1. 생성자에서 함수 목록을 정규화·등록하고 이름 변형(대소문자) 별칭도 함께 등록한다.
2. 내부 `dynamic_invoker` 클로저가 런타임 함수 호출의 진입점 역할을 하며, 이름 조회 → 스키마 검증 → `execute()` 호출 순으로 동작한다.
3. 스키마 조회(`get_*_schema`)는 Pydantic의 `model_json_schema()`를 활용해 JSON Schema를 동적으로 생성한다.
4. `extract_ref_fields()`는 타입 시스템을 직접 검사(`get_origin`, `get_args`, `issubclass`)해 참조 맵을 빌드하며, `JsonCatalog`의 JSON 파싱 방식과 대칭적인 Pydantic 기반 구현이다.
