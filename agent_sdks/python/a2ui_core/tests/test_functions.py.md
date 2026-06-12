# agent_sdks/python/a2ui_core/tests/test_functions.py

## 개요

`BASIC_FUNCTION_IMPLEMENTATIONS`에 등록된 모든 내장 함수(산술, 비교, 논리, 문자열 검증, 값 유효성 검사, 포맷팅, 액션)의 실행 동작을 pytest로 검증하는 테스트 파일이다. 인자를 Pydantic 스키마로 검증한 뒤 구현체를 호출하는 헬퍼 `invoke`를 중심으로 구성되며, 유효 입력의 반환값과 잘못된 입력 시의 `ValidationError` 발생 여부를 함께 확인한다. `formatString`의 동적 값 해석 테스트를 위해 `MockDataContext` 보조 클래스를 내부에서 정의한다.

## 의존성

### 외부 패키지
- `pytest` — 테스트 프레임워크 및 `pytest.raises` 컨텍스트 매니저
- `math` — `math.inf` 상수 (0으로 나누기 결과 검증)
- `typing.Any` — `invoke`와 `MockDataContext` 타입 힌트
- `pydantic.ValidationError` — 잘못된 인자 전달 시 예외 타입

### 저장소 내부 모듈
- [`agent_sdks/python/a2ui_core/src/a2ui/core/basic_catalog/function_impls.py`](../src/a2ui/core/basic_catalog/function_impls.py.md) — `BASIC_FUNCTION_IMPLEMENTATIONS` (내장 함수 구현체 목록)

## Exports

테스트 파일이므로 공개 API 없음. pytest가 수집하는 테스트 함수들과 내부 헬퍼 `invoke`, 보조 클래스 `MockDataContext`, 모듈 수준 상수 `IMPLS_MAP`이 모두 모듈 최상위에 정의된다.

## 상세 명세

### 모듈 수준 상수

**`IMPLS_MAP: dict[str, Any]`**
`BASIC_FUNCTION_IMPLEMENTATIONS` 리스트를 각 구현체의 `name` 속성을 키로 하는 딕셔너리로 변환한 것이다. 모듈 로드 시 한 번만 생성되며, `invoke` 헬퍼에서 이름 기반 조회에 사용된다.

---

### `invoke(name: str, args: dict, context: Any = None) -> Any`

테스트 편의를 위한 함수 실행 래퍼.

1. `IMPLS_MAP`에서 `name`에 해당하는 구현체를 조회한다. 없으면 `ValueError(f"Function {name} not found")`를 발생시킨다.
2. 구현체에 `schema`가 있으면 `impl.schema.model_validate(args).model_dump()`로 인자를 검증하고 직렬화한다. `schema`가 없으면 빈 딕셔너리 `{}`를 사용한다.
3. `impl.execute(validated_args, context)`를 호출하고 결과를 반환한다.

인자 누락이나 타입 오류가 있을 경우 2단계에서 pydantic `ValidationError`가 발생한다.

---

### `MockDataContext`

`formatString` 함수의 데이터 바인딩 및 함수 호출 기능을 시뮬레이션하기 위한 인메모리 컨텍스트 클래스. pytest 픽스처가 아닌 일반 클래스로, 테스트 함수 내에서 직접 인스턴스화한다.

**필드:**
- `data_model: dict` — 경로 기반 바인딩에 사용할 키-값 맵
- `invoker: callable | None` — 함수 호출(`call`) 처리용 콜백. 기본값 `None`.

**`__init__(self, data_model: dict, invoker=None)`**
두 필드를 그대로 저장한다.

**`resolve_dynamic_value(self, part: Any) -> Any`**
- `part`가 `"path"` 키를 가진 딕셔너리이면 `part["path"].lstrip("/")` 경로로 `data_model`을 조회해 반환한다.
- `part`가 `"call"` 키를 가진 딕셔너리이면 `invoker`가 있을 때 `invoker(part["call"], part.get("args", {}))`로 위임하고, 없으면 `ValueError("No invoker for call")`을 발생시킨다.
- 그 외에는 `part`를 그대로 반환한다.

---

## 동작 흐름 (테스트 케이스)

### 산술 (arithmetic)

| 테스트 함수 | 검증 동작 |
|---|---|
| `test_arithmetic_add` | 정수 덧셈(`1+2==3`), 문자열 숫자 강제 변환(`"1"+"2"==3`). `b=None`이나 `b` 미전달 시 `ValidationError`. |
| `test_arithmetic_subtract` | 정수 뺄셈(`5-3==2`). `b=None` 또는 `b` 미전달 시 `ValidationError`. |
| `test_arithmetic_multiply` | 정수 곱셈(`4*2==8`). `b=None` 또는 `b` 미전달 시 `ValidationError`. |
| `test_arithmetic_divide` | 정수 나눗셈(`10/2==5`). `b=0`이면 `math.inf` 반환. 문자열 숫자 허용(`"10"/"2"==5`). `b=None` 또는 `b="invalid"` 시 `ValidationError`. |

### 비교 (comparison)

| 테스트 함수 | 검증 동작 |
|---|---|
| `test_comparison_equals` | 같은 값이면 `True`, 다르면 `False`. `a` 또는 `b` 미전달 시 `ValidationError`. |
| `test_comparison_not_equals` | 다른 값이면 `True`, 같으면 `False`. `a` 미전달 시 `ValidationError`. |
| `test_comparison_greater_than` | `a > b`이면 `True`, 아니면 `False`. `b=None` 또는 `b` 미전달 시 `ValidationError`. |
| `test_comparison_less_than` | `a < b`이면 `True`, 아니면 `False`. `b=None` 또는 `b` 미전달 시 `ValidationError`. |

### 논리 (logical)

| 테스트 함수 | 검증 동작 |
|---|---|
| `test_logical_and` | `values` 리스트가 모두 `True`여야 `True`. 단일 요소 리스트도 정상 처리. |
| `test_logical_or` | `values` 리스트 중 하나라도 `True`이면 `True`. |
| `test_logical_not` | `value`를 반전(`False→True`, `True→False`). 빈 인자 시 `ValidationError`. |

### 문자열 (string)

| 테스트 함수 | 검증 동작 |
|---|---|
| `test_string_contains` | `string`에 `substring`이 포함되면 `True`. `substring` 또는 `string` 미전달 시 `ValidationError`. |
| `test_string_starts_with` | `string`이 `prefix`로 시작하면 `True`. `prefix` 미전달 시 `ValidationError`. |
| `test_string_ends_with` | `string`이 `suffix`로 끝나면 `True`. `suffix` 미전달 시 `ValidationError`. |

### 유효성 검사 (validation)

| 테스트 함수 | 검증 동작 |
|---|---|
| `test_validation_required` | 비어있지 않은 문자열이면 `True`; 빈 문자열 또는 `None`이면 `False`. `value` 미전달 시 `ValidationError`. |
| `test_validation_length` | `min`/`max` 기준으로 문자열 길이를 검증(`"abc"`+`min=2`→`True`, `"abc"`+`max=2`→`False`). `value` 미전달 시 `ValidationError`. |
| `test_validation_numeric` | `min`/`max` 범위 내이면 `True`, 범위 밖이면 `False`. `value` 미전달 시 `ValidationError`. |
| `test_validation_email` | 유효한 이메일(`test@example.com`, 도트·플러스·하이픈 포함 도메인)은 `True`. 단일 레이블 도메인(`test@test`), 1글자 TLD(`test@test.c`), 도메인 없음(`test@.com`), 형식 오류(`invalid`)는 `False`. `value` 미전달 시 `ValidationError`. |
| `test_validation_regex` | 패턴 일치 시 `True`, 불일치 시 `False`. 잘못된 패턴(`"["`)은 `Exception`을 발생시킨다(구현이 `re.error`를 잡지 않음). |

### 포맷팅 (formatting)

| 테스트 함수 | 검증 동작 | 픽스처/모킹 |
|---|---|---|
| `test_formatting_format_string_static` | `${…}` 없는 정적 문자열을 그대로 반환한다. | 없음 |
| `test_formatting_format_string_data_binding` | `${a}` 형식의 경로 바인딩을 `data_model`에서 조회해 치환한다(`"Value: 10"`). | `MockDataContext({"a": 10})` |
| `test_formatting_format_string_function_call` | `${add(a: 5, b: 7)}` 형식의 함수 호출 결과(`12`)를 문자열에 삽입한다(`"Result: 12"`). | `MockDataContext({}, invoker=mock_invoker)` — 로컬 함수로 `add`를 `int(a)+int(b)`로 모킹 |
| `test_formatting_format_string_serialization` | dict는 compact JSON(`{"name":"Alice","age":30}`)으로, list는 compact JSON 배열(`["swift","ios"]`)로 직렬화. `None`은 빈 문자열로 치환(`"val=end"`). `null`을 포함한 리스트는 `[1,null,3]`. | 다양한 `MockDataContext` 인스턴스 |
| `test_formatting_format_number` | `decimals=1`, `grouping=True`(기본)이면 `"1,234.6"`. `grouping=False`이면 `"1234.6"`. | 없음 |
| `test_formatting_format_currency` | `currency="USD"`, `decimals=2`이면 `"$1,234.56"`. 비표준 코드(`"INVALID-CURRENCY"`)는 `"INVALID-CURRENCY 1,234.56"` 형태로 폴백. | 없음 |
| `test_formatting_format_date` | `format="yyyy-MM-dd"`이면 `"2025-01-01"`. `format="ISO"`이면 `"2025-01-01T12:00:00.000Z"`. 잘못된 날짜 문자열은 빈 문자열 반환. | 없음 |
| `test_formatting_pluralize` | `value=1`이면 `one` 값 반환, 그 외에는 `other` 값 반환. `one` 인자 없이 `value=1`이면 `other`로 폴백. | 없음 |

### 액션 (actions)

| 테스트 함수 | 검증 동작 |
|---|---|
| `test_actions_open_url` | `openUrl`은 Python 환경에서 부작용 없이 `None`을 반환한다 (브라우저 전용 기능). |
