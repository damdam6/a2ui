# agent_sdks/python/a2ui_core/src/a2ui/core/catalog/catalog.py

## 개요

A2UI 카탈로그 시스템의 최상위 추상 기반 클래스 두 개(`CatalogApi`, `CatalogImplementation`)를 정의한다. `CatalogApi`는 JSON Schema 조회와 참조 맵 추출에 대한 추상 인터페이스를 제공하고, `CatalogImplementation`은 이를 확장해 런타임 실행(함수 호출, 컴포넌트 클래스 조회)에 필요한 추가 추상 메서드를 더한다. 외부 패키지 의존성이 없는 순수 Python 파일이다.

## 의존성

### 외부 패키지
- `typing` — `Any`, `Dict`, `List`, `Optional`, `Set`, `Tuple`

### 저장소 내부 모듈
없음.

## Exports

| 이름 | 종류 |
|---|---|
| `CatalogApi` | 클래스 (추상 기반) |
| `CatalogImplementation` | 클래스 (추상 확장) |

## 상세 명세

### `CatalogApi`

**역할**: A2UI 카탈로그의 스키마 정보를 노출하는 추상 인터페이스.

#### `__init__(self, spec_version: str, catalog_id: str)`
- `spec_version`이 falsy이면 `ValueError("A2UI specification version must be provided.")` 발생.
- `catalog_id`가 falsy이면 `ValueError("catalog_id must be provided.")` 발생.
- 검증 통과 시 두 값을 `self.spec_version`, `self.catalog_id`로 저장.

#### `get_component_schema(self, comp_type: str) -> Optional[Dict[str, Any]]`
- 주어진 컴포넌트 타입 이름에 해당하는 JSON Schema를 반환하는 추상 메서드.
- 기본 구현: `NotImplementedError("Subclasses must implement get_component_schema()")` 발생.

#### `get_function_schema(self, func_name: str) -> Optional[Dict[str, Any]]`
- 주어진 함수 이름에 해당하는 인수 JSON Schema를 반환하는 추상 메서드.
- 기본 구현: `NotImplementedError` 발생.

#### `get_theme_schema(self) -> Optional[Dict[str, Any]]`
- 카탈로그 테마의 JSON Schema를 반환하는 추상 메서드.
- 기본 구현: `NotImplementedError` 발생.

#### `extract_ref_fields(self) -> Dict[str, Tuple[Set[str], Set[str]]]`
- 카탈로그 내 모든 컴포넌트를 검사해 다른 컴포넌트를 참조하는 필드의 위상 맵을 반환하는 추상 메서드.
- 반환 구조: `{ 컴포넌트명: (단일_참조_필드_집합, 리스트_참조_필드_집합) }`
- 기본 구현: `NotImplementedError` 발생.

---

### `CatalogImplementation(CatalogApi)`

**역할**: `CatalogApi`를 상속해 런타임 실행 능력(컴포넌트/함수 클래스 조회, 함수 직접 호출)을 추가하는 추상 클래스. 서버 측 추론 프롬프트 빌딩뿐 아니라 클라이언트 측 렌더링과 로컬 함수 평가 용도로도 사용된다.

#### `get_component_class(self, comp_type: str) -> Optional[Any]`
- 컴포넌트의 구체 모델 클래스(일반적으로 Pydantic `BaseModel` 서브클래스)를 반환하는 추상 메서드.
- 기본 구현: `NotImplementedError` 발생.

#### `get_function_class(self, func_name: str) -> Optional[Any]`
- 함수 스키마를 표현하는 구체 모델 클래스를 반환하는 추상 메서드.
- 기본 구현: `NotImplementedError` 발생.

#### `get_function_implementation(self, func_name: str) -> Optional[Any]`
- 함수의 구체 `FunctionImplementation` 객체를 반환하는 추상 메서드.
- 기본 구현: `NotImplementedError` 발생.

#### `invoke_function(self, name: str, args: Dict[str, Any], context: Any = None, abort_signal: Optional[Any] = None) -> Any`
- 카탈로그 함수를 동적으로 실행하는 추상 메서드.
- 매개변수: 함수명, 인수 딕셔너리, 실행 컨텍스트(선택), 중단 신호(선택).
- 기본 구현: `NotImplementedError` 발생.

## 동작 흐름

두 클래스 모두 인스턴스화 시 `__init__`에서 필수 인수를 검증하고, 나머지 메서드는 모두 `NotImplementedError`를 발생시킨다. 실제 동작은 `ModelCatalog`(Pydantic 기반) 또는 `JsonCatalog`(JSON 기반) 구체 서브클래스에서 구현된다. 이 파일은 계약(contract) 정의 역할만 수행한다.
