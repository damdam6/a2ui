# agent_sdks/python/a2ui_core/src/a2ui/core/basic_catalog/function_impls.py

## 개요

`function_apis.py`와 `operator_apis.py`에 선언된 API 명세에 실제 실행 로직을 결합하여 `FunctionImplementation` 인스턴스들을 생성하는 파일이다. `create_function_implementation()` 팩토리를 통해 각 API 클래스와 람다 또는 명명 함수를 조합하고, 모든 구현체를 `BASIC_FUNCTION_IMPLEMENTATIONS` 리스트로 집약하여 외부 카탈로그에 등록 가능한 형태로 export한다.

## 의존성

### 외부 패키지
- `re`: 정규식 검증 및 날짜 토큰 치환
- `datetime`: ISO 8601 날짜 파싱 및 포맷팅
- `math`: 무한대(`math.inf`) 및 NaN(`math.nan`) 반환
- `typing`: `Any`, `Dict`, `List`, `Optional`

### 저장소 내부 모듈
- [`../catalog/functions.py`](../catalog/functions.py.md): `create_function_implementation`
- [`./function_apis.py`](./function_apis.py.md): `RequiredApi`, `RegexApi`, `LengthApi`, `NumericApi`, `EmailApi`, `FormatStringApi`, `FormatNumberApi`, `FormatCurrencyApi`, `FormatDateApi`, `PluralizeApi`, `OpenUrlApi`, `AndApi`, `OrApi`, `NotApi`
- [`./operator_apis.py`](./operator_apis.py.md): `AddApi`, `SubtractApi`, `MultiplyApi`, `DivideApi`, `EqualsApi`, `NotEqualsApi`, `GreaterThanApi`, `LessThanApi`, `ContainsApi`, `StartsWithApi`, `EndsWithApi`
- [`./expression_parser.py`](./expression_parser.py.md): `ExpressionParser`

## Exports

### 모듈 수준 상수
| 이름 | 종류 |
|---|---|
| `BASIC_FUNCTION_IMPLEMENTATIONS` | `List[FunctionImplementation]` — 25개 구현체 목록 |

### 개별 `FunctionImplementation` 인스턴스
`RequiredImplementation`, `RegexImplementation`, `LengthImplementation`, `NumericImplementation`, `EmailImplementation`, `FormatStringImplementation`, `FormatNumberImplementation`, `FormatCurrencyImplementation`, `FormatDateImplementation`, `PluralizeImplementation`, `OpenUrlImplementation`, `AndImplementation`, `OrImplementation`, `NotImplementation`, `AddImplementation`, `SubtractImplementation`, `MultiplyImplementation`, `DivideImplementation`, `EqualsImplementation`, `NotEqualsImplementation`, `GreaterThanImplementation`, `LessThanImplementation`, `ContainsImplementation`, `StartsWithImplementation`, `EndsWithImplementation`

## 상세 명세

---

### 내부 헬퍼 함수

#### `_to_float(val: Any) -> float`
`float(val)` 시도. `ValueError` 또는 `TypeError` 발생 시 `ValueError(f"Cannot convert to number: {val}")` 재발생.

#### `_to_bool(val: Any) -> bool`
`bool(val)` 반환. 파이썬 표준 진릿값 변환 규칙 적용.

#### `_to_str(val: Any) -> str`
- `None` → `""`
- `dict` 또는 `list` → `json.dumps(val, separators=(",", ":"))` (공백 없는 최소화 JSON)
- `bool` → `"true"` 또는 `"false"` (파이썬 기본 `str(True)` = `"True"` 대신 소문자 처리)
- 기타 → `str(val)`

---

### 검증 함수 구현체

#### `RequiredImplementation`
`lambda` 기반. `args["value"]`가 `None`, `""`, `[]`이 아니면 `True` 반환.

#### `RegexImplementation`
`lambda` 기반. `re.search(pattern, value)` 결과를 `bool`로 반환. `value`와 `pattern` 모두 `_to_str()`로 변환 후 적용.

#### `LengthImplementation`
`lambda` 기반. `len(_to_str(value)) >= min` (min이 존재할 때) AND `len(_to_str(value)) <= max` (max가 존재할 때) 두 조건의 교집합 반환. `min`/`max`가 `None`이면 해당 방향 제한 없음.

#### `NumericImplementation`
`lambda` 기반. `_to_float(value) >= min` AND `_to_float(value) <= max` — `min`/`max` `None`이면 제한 없음.

#### `EmailImplementation`
`lambda` 기반. 정규식 `^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`를 `re.match()`로 적용해 `bool` 반환.

---

### 문자열/수치 포맷 함수

#### `_format_string(args, context=None, abort_signal=None) -> str`
1. `args["value"]`를 템플릿으로 취득. 비어 있으면 `""` 반환.
2. `ExpressionParser().parse(template)`으로 파트 목록 추출. 파싱 예외 발생 시 원본 `template` 반환.
3. 파트가 없으면 `""` 반환.
4. 각 파트를 순회하며, `context`가 있고 `resolve_dynamic_value` 메서드가 있으면 `context.resolve_dynamic_value(part)` 호출, 없으면 파트 그대로 사용.
5. 모든 파트를 `_to_str()`로 변환 후 `"".join()` 반환.

#### `_format_number(args, context=None, abort_signal=None) -> str`
1. `value`를 `_to_float()` 변환.
2. `decimals`이 존재하면 `f"{{:{',' if grouping else ''}.{int(decimals)}f}}"` 형식 문자열 구성.
3. `decimals`이 없으면 `f"{{:{',' if grouping else ''}f}}"` 사용.
4. `grouping` 기본값은 `True`.
5. Python `str.format()` 호출 결과 반환.

#### `_format_currency(args, context=None, abort_signal=None) -> str`
1. `value`를 `_to_float()`, `currency` 취득 (기본 `"USD"`).
2. `decimals` 기본값 2, `grouping` 기본값 `True`.
3. 통화 기호: `currency == "USD"` → `"$"`, 그 외 → `currency + " "` (예: `"EUR "`).
4. `symbol + fmt_str.format(val)` 반환.

#### `_format_date(args, context=None, abort_signal=None) -> str`
1. `value`가 없으면 `""` 반환.
2. `datetime.datetime.fromisoformat(str(val).replace("Z", "+00:00"))`으로 파싱. 실패 시 `""` 반환.
3. `fmt == "ISO"` 시: `dt.isoformat().replace("+00:00", ".000Z")` 반환.
4. 그 외: 모듈 수준 `_DATE_TOKENS` 정규식으로 포맷 문자열을 Python `strftime` 토큰으로 변환 후 `dt.strftime(py_fmt)` 반환.

`_DATE_TOKENS` 정규식: `r"yyyy|yy|MMMM|MMM|MM|M|EEEE|E|dd|d|HH|H|hh|h|mm|ss|a|%"` (긴 토큰 우선)

`_DATE_MAP` 변환표:

| 입력 토큰 | Python strftime |
|---|---|
| `%` | `%%` |
| `yyyy` | `%Y` |
| `yy` | `%y` |
| `MMMM` | `%B` |
| `MMM` | `%b` |
| `MM`, `M` | `%m` |
| `EEEE` | `%A` |
| `E` | `%a` |
| `dd`, `d` | `%d` |
| `HH`, `H` | `%H` |
| `hh`, `h` | `%I` |
| `mm` | `%M` |
| `ss` | `%S` |
| `a` | `%p` |

#### `_pluralize(args, context=None, abort_signal=None) -> str`
1. `value`를 `_to_float()` 변환.
2. 카테고리 결정: `0` → `"zero"`, `1` → `"one"`, `2` → `"two"`, 그 외 → `"other"`.
3. `args.get(category) or args.get("other") or ""` 반환.
4. 참고: `"few"`, `"many"` 카테고리는 API에 정의되어 있으나 이 구현체는 숫자 값으로 직접 매핑하는 단순화 로직 사용.

#### `OpenUrlImplementation`
`lambda` 기반. 항상 `None` 반환 (실제 URL 열기는 클라이언트 측 렌더러가 처리).

---

### 논리 연산자

#### `AndImplementation`
`all(_to_bool(v) for v in args["values"])` — 모든 값이 참이면 `True`.

#### `OrImplementation`
`any(_to_bool(v) for v in args["values"])` — 하나 이상 참이면 `True`.

#### `NotImplementation`
`not _to_bool(args["value"])`.

---

### 산술 연산자

모든 산술 연산은 결과가 정수 가능하면 (`res.is_integer()`) `int(res)` 반환, 아니면 `float` 반환.

#### `_add(args, context=None, abort_signal=None)`
`_to_float(args["a"]) + _to_float(args["b"])`.

#### `_subtract(args, context=None, abort_signal=None)`
`_to_float(args["a"]) - _to_float(args["b"])`.

#### `_multiply(args, context=None, abort_signal=None)`
`_to_float(args["a"]) * _to_float(args["b"])`.

#### `_divide(args, context=None, abort_signal=None)`
`b == 0` 처리:
- `a > 0` → `math.inf`
- `a < 0` → `-math.inf`
- `a == 0` → `math.nan`

`b != 0`이면 `a / b` 계산 후 정수 가능 시 `int` 반환.

---

### 비교 연산자

#### `EqualsImplementation`
`args.get("a") == args.get("b")` — 파이썬 동등 비교.

#### `NotEqualsImplementation`
`args.get("a") != args.get("b")`.

#### `GreaterThanImplementation`
`_to_float(args["a"]) > _to_float(args["b"])`.

#### `LessThanImplementation`
`_to_float(args["a"]) < _to_float(args["b"])`.

---

### 문자열 연산자

#### `ContainsImplementation`
`_to_str(args["substring"]) in _to_str(args["string"])` — Python `in` 연산자 사용.

#### `StartsWithImplementation`
`_to_str(args["string"]).startswith(_to_str(args["prefix"]))`.

#### `EndsWithImplementation`
`_to_str(args["string"]).endswith(_to_str(args["suffix"]))`.

---

### `BASIC_FUNCTION_IMPLEMENTATIONS`
타입: `List[FunctionImplementation]`

25개의 `FunctionImplementation` 인스턴스를 순서대로 담은 리스트. 순서:
`Required`, `Regex`, `Length`, `Numeric`, `Email`, `FormatString`, `FormatNumber`, `FormatCurrency`, `FormatDate`, `Pluralize`, `OpenUrl`, `And`, `Or`, `Not`, `Add`, `Subtract`, `Multiply`, `Divide`, `Equals`, `NotEquals`, `GreaterThan`, `LessThan`, `Contains`, `StartsWith`, `EndsWith`.

## 동작 흐름

1. 모듈 임포트 시 `create_function_implementation(Api, execute_fn)` 호출이 순서대로 평가된다.
2. 각 호출은 `FunctionImplementation`을 상속하는 동적 클래스를 즉시 인스턴스화하여 반환한다. 반환된 인스턴스는 모듈 수준 변수(`RequiredImplementation` 등)에 바인딩된다.
3. `BASIC_FUNCTION_IMPLEMENTATIONS` 리스트가 25개 인스턴스를 집약한다.
4. 외부 카탈로그 구성 코드가 이 리스트를 순회하며 `name`으로 함수를 등록하고, 런타임에 `execute(args, context, abort_signal)`를 호출하여 함수를 실행한다.
5. `_format_string` 실행 시에만 `ExpressionParser`가 동적으로 인스턴스화되어 템플릿을 파싱하고, `context.resolve_dynamic_value()`를 통해 데이터 바인딩 파트를 해석한다.
