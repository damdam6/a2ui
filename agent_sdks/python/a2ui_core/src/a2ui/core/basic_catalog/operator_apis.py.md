# agent_sdks/python/a2ui_core/src/a2ui/core/basic_catalog/operator_apis.py

## 개요

기본 카탈로그에서 사용하는 산술·비교·문자열 연산 함수들의 API 명세를 정의한다. TypeScript 측 `renderers/web_core/src/v0_9/basic_catalog/functions/basic_functions_api.ts`를 Python으로 미러링한 파일이다. 각 연산자는 인수(Args) 모델 클래스와 API 클래스 쌍으로 구성되며, 함수 이름·인수 타입·반환 타입을 선언적으로 기술한다.

## 의존성

### 외부 패키지
- `typing` — `Any`, `Dict`, `List`, `Optional`
- `pydantic` — `Field`

### 저장소 내부 모듈
- [`../schema/common_types.py`](../schema/common_types.py.md) — `StrictBaseModel`, `DynamicString`, `DynamicNumber`, `DynamicBoolean`, `DynamicValue`, `DynamicStringList`
- [`../catalog/functions.py`](../catalog/functions.py.md) — `FunctionApi`

## Exports

모든 클래스가 공개 심볼로 노출된다 (`__all__` 미선언, 직접 import 가능).

| 클래스 | 종류 |
|---|---|
| `AddArgs` | 클래스 (Pydantic 모델) |
| `AddApi` | 클래스 (FunctionApi 하위) |
| `SubtractArgs` | 클래스 (Pydantic 모델) |
| `SubtractApi` | 클래스 (FunctionApi 하위) |
| `MultiplyArgs` | 클래스 (Pydantic 모델) |
| `MultiplyApi` | 클래스 (FunctionApi 하위) |
| `DivideArgs` | 클래스 (Pydantic 모델) |
| `DivideApi` | 클래스 (FunctionApi 하위) |
| `EqualsArgs` | 클래스 (Pydantic 모델) |
| `EqualsApi` | 클래스 (FunctionApi 하위) |
| `NotEqualsArgs` | 클래스 (Pydantic 모델) |
| `NotEqualsApi` | 클래스 (FunctionApi 하위) |
| `GreaterThanArgs` | 클래스 (Pydantic 모델) |
| `GreaterThanApi` | 클래스 (FunctionApi 하위) |
| `LessThanArgs` | 클래스 (Pydantic 모델) |
| `LessThanApi` | 클래스 (FunctionApi 하위) |
| `ContainsArgs` | 클래스 (Pydantic 모델) |
| `ContainsApi` | 클래스 (FunctionApi 하위) |
| `StartsWithArgs` | 클래스 (Pydantic 모델) |
| `StartsWithApi` | 클래스 (FunctionApi 하위) |
| `EndsWithArgs` | 클래스 (Pydantic 모델) |
| `EndsWithApi` | 클래스 (FunctionApi 하위) |

## 상세 명세

### 패턴

파일 전체는 동일한 패턴을 반복한다. 각 연산자마다:
1. `*Args` 클래스 — `StrictBaseModel` 상속, 인수 필드를 `Field(...)` (필수)로 선언
2. `*Api` 클래스 — `FunctionApi` 상속, 클래스 변수 `name`, `args`, `return_type` 선언

`FunctionApi.__init_subclass__`가 서브클래스 정의 시점에 `name`과 `schema`(=`args`)를 자동 등록한다.

---

### 산술 연산 (반환 타입: `"number"`)

#### `AddArgs`
- 필드 `a: DynamicNumber = Field(...)`, `b: DynamicNumber = Field(...)` — 두 피연산자. `DynamicNumber`는 `float | DataBinding | FunctionCall`의 Union이므로 리터럴 숫자, 데이터 바인딩, 함수 호출 모두 허용.

#### `AddApi`
- `name = "add"`, `args = AddArgs`, `return_type = "number"`

#### `SubtractArgs`
- 필드 `a: DynamicNumber`, `b: DynamicNumber` — `AddArgs`와 동일 구조.

#### `SubtractApi`
- `name = "subtract"`, `args = SubtractArgs`, `return_type = "number"`

#### `MultiplyArgs`
- 필드 `a: DynamicNumber`, `b: DynamicNumber`

#### `MultiplyApi`
- `name = "multiply"`, `args = MultiplyArgs`, `return_type = "number"`

#### `DivideArgs`
- 필드 `a: DynamicNumber`, `b: DynamicNumber`

#### `DivideApi`
- `name = "divide"`, `args = DivideArgs`, `return_type = "number"`

---

### 동등성 비교 연산 (반환 타입: `"boolean"`)

#### `EqualsArgs`
- 필드 `a: Any = Field(...)`, `b: Any = Field(...)` — 피연산자 타입 제한 없음. 임의 값 비교 허용.

#### `EqualsApi`
- `name = "equals"`, `args = EqualsArgs`, `return_type = "boolean"`

#### `NotEqualsArgs`
- 필드 `a: Any`, `b: Any` — `EqualsArgs`와 동일 구조.

#### `NotEqualsApi`
- `name = "not_equals"`, `args = NotEqualsArgs`, `return_type = "boolean"`

---

### 대소 비교 연산 (반환 타입: `"boolean"`)

#### `GreaterThanArgs`
- 필드 `a: DynamicNumber`, `b: DynamicNumber`

#### `GreaterThanApi`
- `name = "greater_than"`, `args = GreaterThanArgs`, `return_type = "boolean"`

#### `LessThanArgs`
- 필드 `a: DynamicNumber`, `b: DynamicNumber`

#### `LessThanApi`
- `name = "less_than"`, `args = LessThanArgs`, `return_type = "boolean"`

---

### 문자열 연산 (반환 타입: `"boolean"`)

#### `ContainsArgs`
- 필드 `string: DynamicString = Field(...)` — 검색 대상 문자열. `DynamicString`은 `str | DataBinding | FunctionCall`.
- 필드 `substring: DynamicString = Field(...)` — 찾을 부분 문자열.

#### `ContainsApi`
- `name = "contains"`, `args = ContainsArgs`, `return_type = "boolean"`

#### `StartsWithArgs`
- 필드 `string: DynamicString`, `prefix: DynamicString` — 대상 문자열과 접두사.

#### `StartsWithApi`
- `name = "starts_with"`, `args = StartsWithArgs`, `return_type = "boolean"`

#### `EndsWithArgs`
- 필드 `string: DynamicString`, `suffix: DynamicString` — 대상 문자열과 접미사.

#### `EndsWithApi`
- `name = "ends_with"`, `args = EndsWithArgs`, `return_type = "boolean"`

## 동작 흐름

이 파일은 런타임 로직을 포함하지 않는다. 선언만 존재하며, 임포트 시점에 `FunctionApi.__init_subclass__`에 의해 각 `*Api` 클래스의 `name`, `schema` 클래스 변수가 자동으로 채워진다. `ModelCatalog` 등이 이 클래스들을 `functions` 딕셔너리에 등록해 사용한다.
