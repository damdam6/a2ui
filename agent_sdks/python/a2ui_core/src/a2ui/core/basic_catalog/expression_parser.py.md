# agent_sdks/python/a2ui_core/src/a2ui/core/basic_catalog/expression_parser.py

## 개요

`${...}` 템플릿 문자열 표현식을 파싱하는 렉서·파서 구현 파일이다. `Scanner` 클래스가 문자 수준의 커서 탐색을 담당하고, `ExpressionParser` 클래스가 그 위에서 재귀 하강 방식으로 리터럴·경로 참조·함수 호출을 인식해 구조화된 파이썬 객체로 변환한다. 외부 패키지 의존성은 표준 라이브러리 `re`만 존재하며, 다른 저장소 내부 모듈을 임포트하지 않는다.

## 의존성

### 외부 패키지
- `re`: 키워드 종료 경계 확인에 사용
- `typing`: `Any`, `Dict`, `List`, `Union`

### 저장소 내부 모듈
없음

## Exports

| 이름 | 종류 |
|---|---|
| `Scanner` | 클래스 |
| `ExpressionParser` | 클래스 |

## 상세 명세

---

### `Scanner`

문자열을 한 글자씩 탐색하는 커서 기반 렉서 헬퍼. 상태로 `input: str`(원본 문자열)과 `pos: int`(현재 위치)를 유지한다.

#### `__init__(self, input_str: str)`
- `self.input = input_str`, `self.pos = 0`으로 초기화.

#### `is_at_end(self) -> bool`
- `self.pos >= len(self.input)`이면 `True` 반환.

#### `peek(self, offset: int = 0) -> str`
- `self.pos + offset` 위치의 문자를 반환. 범위를 벗어나면 `"\0"` 반환. 커서를 이동하지 않는다.

#### `advance(self, count: int = 1) -> str`
- `self.pos`부터 `count`글자를 슬라이싱해 반환하고 `self.pos`를 `count`만큼 전진. 항상 소비된 문자열을 반환한다.

#### `match(self, expected: str) -> bool`
- 현재 위치가 `expected` 단일 문자이면 `advance()`하고 `True` 반환, 아니면 `False` 반환.

#### `matches(self, expected: str) -> bool`
- `self.input.startswith(expected, self.pos)` — 현재 위치에서 문자열 `expected`로 시작하는지 확인. 커서 이동 없음.

#### `matches_string(self, expected: str) -> bool`
- `peek() == expected`와 동일. 현재 문자가 주어진 단일 문자인지 확인.

#### `matches_keyword(self, keyword: str) -> bool`
- 현재 위치에서 `keyword`로 시작하고, `keyword` 직후 문자가 알파뉴메릭/언더스코어(`[a-zA-Z0-9_]`)가 아닐 때 `advance(len(keyword))`하고 `True` 반환. 그렇지 않으면 커서 이동 없이 `False` 반환. 키워드 경계를 보장한다.

#### `skip_whitespace(self)`
- `isspace()`를 만족하는 문자를 모두 소비해 건너뜀.

---

### `ExpressionParser`

재귀 하강 파서. `${...}` 보간이 포함된 문자열을 파트 목록으로 분해하거나, 단일 표현식 문자열을 파이썬 값/구조체로 변환한다.

#### 클래스 상수
- `MAX_DEPTH: int = 10` — 재귀 최대 깊이. 초과 시 `ValueError` 발생.

---

#### `parse(self, input_str: str, depth: int = 0) -> List[Any]`
최상위 진입점. 템플릿 문자열 `input_str`에서 `${...}` 보간 구간과 일반 텍스트를 순서대로 추출해 파트 리스트로 반환한다.

동작 단계:
1. `depth > MAX_DEPTH`이면 `ValueError("Max recursion depth reached in parse")` 발생.
2. `input_str`이 비어 있거나 `"${"` 부분 문자열이 없으면, 비어 있지 않은 경우 `[input_str]`, 비어 있으면 `[]` 즉시 반환.
3. `Scanner(input_str)`로 스캐너 생성 후 끝에 도달할 때까지 반복:
   - `"${"` 매칭 시: 2글자 소비 후 `extract_interpolation_content()`로 중괄호 내용 추출, `parse_expression(content, depth+1)`로 파싱. 결과가 `None` 또는 `""` 아니면 `parts`에 추가.
   - `\${` 이스케이프 시(`\` + `$` + `{` 연속): `\` 1글자 소비 후 리터럴 `"${"` 문자열을 `parts`에 추가하고 2글자 소비.
   - 그 외: `"${"` 또는 `\${`를 만날 때까지 글자 단위로 전진하여 텍스트 구간을 슬라이싱해 `parts`에 추가.
4. `None`과 `""` 파트를 필터링한 리스트 반환.

---

#### `extract_interpolation_content(self, scanner: Scanner) -> str`
`${` 직후부터 매칭 `}` 직전까지의 내용 문자열을 반환한다. `scanner.pos`는 이미 `${` 뒤를 가리키고 있다고 가정.

동작 단계:
1. `start = scanner.pos`, `brace_balance = 1`.
2. `brace_balance > 0`인 동안 반복:
   - `{` → `brace_balance += 1`.
   - `}` → `brace_balance -= 1`.
   - `'` 또는 `"` → 해당 따옴표 쌍을 완전히 소비 (이스케이프 `\` 처리 포함). 내부의 중괄호는 brace 카운트에 영향 안 줌.
3. 루프 종료 후 `brace_balance > 0`이면 `ValueError("Unclosed interpolation: missing '}'")`.
4. `scanner.input[start : scanner.pos - 1]` 반환 (마지막 `}` 제외).

---

#### `parse_expression(self, expr: str, depth: int = 0) -> Any`
단독 표현식 문자열을 파싱해 파이썬 값 반환. 외부에서 표현식 하나를 직접 평가할 때 사용.

동작 단계:
1. `depth > MAX_DEPTH` → `ValueError`.
2. `expr.strip()` 후 빈 문자열이면 `""` 반환.
3. `Scanner(expr)` 생성 후 `_parse_expression_internal(scanner, depth)` 호출.
4. 파싱 후 `scanner.is_at_end()`가 아니면 `ValueError(f"Unexpected characters at end of expression: '...'")`.
5. 결과 반환.

---

#### `_parse_expression_internal(self, scanner: Scanner, depth: int) -> Any`
내부 재귀 파서 핵심 함수. 현재 커서 위치를 읽어 토큰 종류를 판별한 뒤 적절한 파싱 메서드로 위임.

판별 순서 (우선순위 순):
0. **중첩 보간**: `"${"` → 2글자 소비, `extract_interpolation_content()`, `parse_expression(content, depth+1)` 재귀 반환.
1. **문자열 리터럴**: `'` 또는 `"` → `parse_string_literal()` 반환.
2. **숫자 리터럴**: 현재 문자가 숫자이거나 `-` + 다음 숫자 → `parse_number_literal()` 반환.
3. **`true`** → `True` 반환.
4. **`false`** → `False` 반환.
5. **`null`** → `""` 반환 (null을 빈 문자열로 매핑).
6. **식별자/경로**: `scan_path_or_identifier()` 호출 → 공백 건너뜀 후:
   - 다음 문자가 `(` → `parse_function_call(token, scanner, depth)` 반환.
   - 그 외 → `{"path": token}` 딕셔너리 반환. `token`이 빈 문자열이면 `""` 반환.

---

#### `scan_path_or_identifier(self, scanner: Scanner) -> str`
알파뉴메릭, `/`, `.`, `_`, `-` 문자가 연속되는 구간을 소비하고 해당 문자열 반환. 경로(`a/b.c`) 및 식별자 모두 인식.

---

#### `parse_function_call(self, func_name: str, scanner: Scanner, depth: int) -> Dict[str, Any]`
`(` 직후부터 `)` 까지 `name: value` 형식의 인수를 파싱.

동작 단계:
1. `(` 소비 및 공백 건너뜀.
2. `)` 아닌 동안 반복:
   - `scan_identifier()`로 인수 이름 스캔.
   - `:` 소비 (없으면 `ValueError`).
   - `_parse_expression_internal()`로 값 재귀 파싱 → `args[arg_name]`에 저장.
   - `,` 있으면 소비 및 공백 건너뜀.
3. `)` 소비 (없으면 `ValueError`).
4. `{"call": func_name, "args": args, "returnType": "any"}` 반환.

---

#### `scan_identifier(self, scanner: Scanner) -> str`
알파뉴메릭 또는 `_`만 허용하는 식별자 스캔. `scan_path_or_identifier`와 달리 `/`, `.`, `-` 미포함.

---

#### `parse_string_literal(self, scanner: Scanner) -> str`
`'` 또는 `"` 따옴표 하나 소비 후 닫는 따옴표까지 문자열 파싱. 이스케이프 처리:
- `\n` → 개행, `\t` → 탭, `\r` → 캐리지 리턴, `\x` (기타) → `x` 그대로.

---

#### `parse_number_literal(self, scanner: Scanner) -> Union[int, float]`
선택적 `-` 뒤에 숫자와 소수점으로 이루어진 토큰을 소비. `.` 포함이면 `float`, 아니면 `int` 반환.

---

#### `is_alnum(self, c: str) -> bool`
`a-z`, `A-Z`, `0-9` 범위 확인 (표준 라이브러리 미사용).

#### `is_digit(self, c: str) -> bool`
`0-9` 범위 확인.

## 동작 흐름

1. 외부에서 `ExpressionParser().parse("Hello ${name}!")` 형태로 호출.
2. `parse()`가 입력 문자열을 스캔하며 `"Hello "`, `{"path": "name"}`, `"!"` 파트 3개로 분리.
3. `${...}` 내용은 `extract_interpolation_content()`로 추출되고 `parse_expression()` → `_parse_expression_internal()`로 내려간다.
4. 내부 파서는 현재 토큰이 리터럴인지, 경로 참조인지, 함수 호출인지를 순서대로 판별해 파이썬 구조체(`str`, `int`, `float`, `bool`, `{"path":...}`, `{"call":..., "args":..., "returnType":"any"}`)로 변환.
5. 중첩 `${...}`는 재귀 호출로 처리되며 `MAX_DEPTH = 10` 초과 시 예외 발생.
