# agent_sdks/python/a2ui_core/src/a2ui/core/catalog/functions.py

## 개요

카탈로그 함수의 명세(`FunctionApi`)와 실행 가능한 구현(`FunctionImplementation`)을 정의하고, 클로저에서 동적으로 구현체를 생성하는 헬퍼 함수(`create_function_implementation`)를 제공한다. 또한 함수 호출 시그니처를 표현하는 타입 별칭 `FunctionInvoker`를 선언한다. 자동 생성된 파일이다.

## 의존성

### 외부 패키지
- `typing` — `Any`, `Callable`, `Dict`, `Optional`, `Type`

### 저장소 내부 모듈
없음.

## Exports

| 이름 | 종류 |
|---|---|
| `FunctionApi` | 클래스 |
| `FunctionImplementation` | 클래스 |
| `create_function_implementation` | 함수 |
| `FunctionInvoker` | 타입 별칭 |

## 상세 명세

### `FunctionApi`

함수 스키마 명세를 보관하는 기반 클래스. 서브클래스는 클래스 변수로 `name`, `return_type`, `args`(또는 `schema`)를 선언해 명세를 정의한다.

#### 클래스 변수 (기본값)
- `name: str = ""`
- `return_type: str = "any"`
- `schema: Any = None`

#### `__init__(self, name=None, return_type=None, schema=None)`
인스턴스 생성 시:
1. `name` 인수가 없으면 클래스의 `name` 속성 조회.
2. `return_type` 인수가 없으면 클래스의 `return_type`, 없으면 `returnType` 순으로 조회, 없으면 `"any"`.
3. `schema` 인수가 없으면 클래스의 `schema`, 없으면 `args` 순으로 조회.

#### `__init_subclass__(cls, **kwargs)`
서브클래스 정의 시점에 자동 호출되는 훅:
1. `cls.name`이 없으면 `cls.call` 속성으로 대체.
2. `cls.return_type`이 없으면 `cls.returnType`으로 대체.
3. `cls.schema`가 없으면 `cls.args`를 시도하고, 그마저도 없으면 `cls.__annotations__`에서 `"args"` 키를 찾아 할당. 이를 통해 서브클래스가 `args = SomeModel`처럼 선언해도 `schema`로 자동 등록된다.

---

### `FunctionImplementation(FunctionApi)`

`FunctionApi`에 실행 로직을 추가한 추상 클래스.

#### `__init__(self, name: str, return_type: str, schema: Any)`
부모 `FunctionApi.__init__`을 `name`, `return_type`, `schema`를 명시적으로 전달해 호출한다.

#### `execute(self, args: Dict[str, Any], context: Any = None, abort_signal: Optional[Any] = None) -> Any`
검증·강제변환된 인수로 함수를 실행하는 추상 메서드. 기본 구현: `NotImplementedError("Subclasses must override and implement execute()")` 발생.

---

### `create_function_implementation(api: Any, execute: Callable[[Dict[str, Any], Any, Optional[Any]], Any]) -> FunctionImplementation`

API 명세 객체(또는 클래스)와 실행 클로저를 조합해 즉석에서 `FunctionImplementation` 인스턴스를 생성하는 팩토리 함수.

**동작 단계:**
1. 함수 내부에 `DynamicFunctionImplementation(FunctionImplementation)` 로컬 클래스를 정의한다.
2. 해당 클래스의 `__init__`에서 `api` 인수로부터 `name`, `return_type`, `schema`를 다단계 `getattr` 조회로 추출한다.
   - `name`: `api.name` → `api.__class__.name` 순으로 시도.
   - `return_type`: `api.return_type` → `api.returnType` → `api.__class__.return_type` → `api.__class__.returnType` → `"any"` 순.
   - `schema`: `api.schema` → `api.args` → `api.__class__.schema` → `api.__class__.args` → `None` 순.
3. `execute` 메서드는 전달받은 클로저를 그대로 위임 호출(`return execute(args, context, abort_signal)`)한다.
4. `DynamicFunctionImplementation()`를 인스턴스화해 반환한다.

---

### `FunctionInvoker`

`Callable[[str, Dict[str, Any], Any, Optional[Any]], Any]`

함수 이름, 인수 딕셔너리, 컨텍스트, 중단 신호를 받아 임의 값을 반환하는 호출 가능 객체의 타입 별칭. `ModelCatalog`의 `invoker` 필드 타입으로 사용된다.

## 동작 흐름

1. `FunctionApi` 서브클래스를 정의하면 `__init_subclass__`가 즉시 `name`과 `schema`를 채운다.
2. 인스턴스화 시 `__init__`이 클래스 변수를 인스턴스 변수로 복사한다.
3. 실행 로직이 필요한 경우 `FunctionImplementation`을 상속하거나 `create_function_implementation`으로 클로저를 결합한다.
4. 최종적으로 `FunctionInvoker` 타입의 `invoker` 함수를 통해 외부에서 이름 기반으로 호출한다.
