# agent_sdks/python/a2ui_core/tests/test_expression_parser.py

## 개요

`ExpressionParser` 클래스의 파싱 동작을 검증하는 pytest 테스트 파일이다. 리터럴 문자열, `${...}` 보간 표현식, 함수 호출 구문, 키워드(`true`/`false`/`null`), 이스케이프 시퀀스, 경계 조건(최대 재귀 깊이, 닫히지 않은 보간, 잘못된 함수 문법, 예상치 못한 문자) 등 다양한 입력 유형에 대해 `parse()`와 `parse_expression()` 메서드의 반환값 및 예외 발생을 검증한다.

## 의존성

### 외부 패키지
- `pytest`

### 저장소 내부 모듈
- [`agent_sdks/python/a2ui_core/src/a2ui/core/basic_catalog/expression_parser.py`](../src/a2ui/core/basic_catalog/expression_parser.py.md) — `ExpressionParser`

## Exports

pytest 픽스처(`parser`)와 테스트 함수만 정의하며 외부로 export되는 공개 심벌은 없다.

## 상세 명세 (픽스처 및 테스트 케이스)

### 픽스처

#### `parser()` → `ExpressionParser`
`ExpressionParser()`를 생성하여 반환하는 함수 스코프 pytest 픽스처. `@pytest.fixture` 데코레이터로 정의된다. 모든 테스트 함수가 이 픽스처를 매개변수로 주입받아 사용한다. 함수 스코프이므로 각 테스트마다 새 인스턴스가 생성된다.

---

### 테스트 케이스

#### `test_parses_literal_strings_unchanged(parser)`
**검증 동작:** 보간 없는 순수 리터럴 문자열 `"hello world"`를 `parse()`에 전달하면 단일 문자열 원소 `["hello world"]`를 반환해야 한다.

#### `test_parses_simple_interpolation(parser)`
**검증 동작:** `"hello ${foo}"`를 파싱하면 리터럴 세그먼트와 path 참조 객체가 결합된 `["hello ", {"path": "foo"}]`를 반환해야 한다.

#### `test_parses_number_interpolation(parser)`
**검증 동작:** `"number is ${num}"`를 파싱하면 `["number is ", {"path": "num"}]`를 반환해야 한다.

#### `test_parses_nested_interpolation(parser)`
**검증 동작:** `"val is ${${nested}}"`처럼 보간 안에 보간이 있는 경우 `["val is ", {"path": "nested"}]`를 반환해야 한다. 내부 `${nested}`가 먼저 `{"path": "nested"}`로 해석되고, 외부 보간은 그 결과를 그대로 통합한다.

#### `test_handles_escaped_interpolation(parser)`
**검증 동작:** `"escaped \\${foo}"`처럼 `\$`로 이스케이프된 보간 시퀀스는 그대로 텍스트로 취급되어 `["escaped ", "${", "foo}"]`를 반환해야 한다. `\$`가 `"${"` 리터럴 문자열로 분리됨을 확인한다.

#### `test_parses_function_calls(parser)`
**검증 동작:** `"sum is ${add(a: 10, b: 20)}"`를 파싱하면 숫자 인수가 올바르게 파싱된 함수 호출 객체를 포함한 리스트를 반환해야 한다:
- 기대값: `["sum is ", {"call": "add", "args": {"a": 10, "b": 20}, "returnType": "any"}]`

#### `test_parses_function_calls_with_string_literals(parser)`
**검증 동작:** `'case is ${upper(text: "hello")}'`를 파싱하면 쌍따옴표 문자열 인수가 Python 문자열로 파싱된 결과를 반환해야 한다:
- 기대값: `["case is ", {"call": "upper", "args": {"text": "hello"}, "returnType": "any"}]`

#### `test_parses_keywords(parser)`
**검증 동작:** `"${true} ${false} ${null}"`를 파싱하면 `[True, " ", False, " "]`를 반환해야 한다. `true` → `True`, `false` → `False`로 변환되고, `null`은 `None`으로 변환되지만 결과 리스트에 포함되지 않아 최종 결과는 4개 원소가 된다.

#### `test_returns_error_on_max_depth_exceeded(parser)`
**검증 동작:** `parser.parse("depth", 11)`처럼 두 번째 인수로 11(최대 재귀 깊이 초과 값)을 전달하면 `"Max recursion depth reached"` 메시지의 `ValueError`가 발생해야 한다.

#### `test_handles_deep_recursion_gracefully(parser)`
**검증 동작:** `'${${"hello"}}'`처럼 중첩 깊이가 허용 범위 내인 경우 `["hello"]`를 반환해야 한다. 내부 `${"hello"}`가 문자열 리터럴 `"hello"`로 평가된 뒤 외부 보간 결과로 통합된다.

#### `test_returns_error_on_unclosed_interpolation(parser)`
**검증 동작:** `"hello ${world"`처럼 `${`가 닫히지 않은 경우 `"Unclosed interpolation"` 메시지의 `ValueError`가 발생해야 한다.

#### `test_returns_error_on_invalid_function_syntax(parser)`
**검증 동작:** `"${add(a: 1, b: 2}"` 처럼 함수 호출의 닫는 괄호 `)` 대신 `}`가 오는 잘못된 문법에서 `"Expected '\\)'"` 메시지의 `ValueError`가 발생해야 한다.

#### `test_returns_error_on_unexpected_characters_at_end(parser)`
**검증 동작:** `"${true false}"` 처럼 보간 내부에서 토큰 파싱 후 예상치 못한 추가 문자가 있을 때 `"Unexpected characters"` 메시지의 `ValueError`가 발생해야 한다.

#### `test_handles_empty_identifiers(parser)`
**검증 동작:** 빈 식별자와 빈 함수 호출에 대한 세 가지 동작을 검증한다:
1. `parser.parse("${()}")` → `[{"call": "", "args": {}, "returnType": "any"}]`
2. `parser.parse_expression("")` → `""` (빈 문자열)
3. `parser.parse_expression("()")` → `{"call": "", "args": {}, "returnType": "any"}`

#### `test_handles_string_literals_with_escaped_characters(parser)`
**검증 동작:** `parse_expression()`에 raw 문자열 `r"'line1\nline2\t\r\'\\x'"`를 전달하면 이스케이프 시퀀스가 처리된 Python 문자열 `"line1\nline2\t\r'\\x"`를 반환해야 한다. `\n`, `\t`, `\r`, `\'`, `\\` 이스케이프 시퀀스 각각이 올바르게 처리됨을 확인한다.

#### `test_handles_parsing_paths_with_special_characters(parser)`
**검증 동작:** `parse_expression("my-path.with_underscores")`를 호출하면 `{"path": "my-path.with_underscores"}`를 반환해야 한다. 하이픈(`-`)과 언더스코어(`_`)가 path 식별자 내에서 허용됨을 확인한다.

#### `test_returns_error_on_missing_colon_in_function_args(parser)`
**검증 동작:** `parse_expression("add(a 10, b: 20)")`처럼 함수 인수에서 `:`가 누락된 경우 `"Expected ':'"` 메시지의 `ValueError`가 발생해야 한다.

## 동작 흐름

파일 최상단에 `@pytest.fixture`로 정의된 `parser()` 픽스처가 `ExpressionParser` 인스턴스를 제공한다. 모든 테스트 함수는 이 픽스처를 매개변수로 받아 동일 인터페이스의 새 인스턴스를 사용한다. 테스트는 두 가지 패턴으로 동작한다: `parser.parse(string)` 또는 `parser.parse_expression(string)`의 반환값을 기대값과 직접 비교(`==`)하거나, `pytest.raises(ValueError)`로 특정 에러 메시지 패턴을 검증한다. 정상 경로 테스트(리터럴, 보간, 함수 호출, 키워드, 이스케이프, 재귀)와 오류 경로 테스트(최대 깊이 초과, 닫히지 않은 보간, 잘못된 문법, 예상치 못한 문자, 누락된 콜론)가 균형 있게 분포한다.
