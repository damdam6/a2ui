# renderers/web_core/src/v0_9/basic_catalog/expressions/expression_parser.test.ts

## 개요

`ExpressionParser` 클래스의 동작을 검증하는 단위 테스트 파일이다. Node.js 내장 테스트 러너(`node:test`, `node:assert`)를 사용하며, `beforeEach`에서 매 테스트마다 새로운 `ExpressionParser` 인스턴스를 생성하는 픽스처 패턴을 사용한다. `parse`와 `parseExpression` 두 공개 메서드의 정상 동작과 에러 케이스를 포괄적으로 검증한다.

## 의존성

### 외부 패키지
- `node:test` — `describe`, `it`, `beforeEach` 임포트.
- `node:assert` — `assert.deepStrictEqual`, `assert.throws` 임포트.

### 저장소 내부 모듈
- [`./expression_parser.ts`](./expression_parser.ts.md) — `ExpressionParser` 클래스 임포트.

## Exports

없음. 테스트 파일이므로 외부로 내보내는 항목이 없다.

## 픽스처 / 모킹

- **픽스처**: `beforeEach`에서 `parser = new ExpressionParser()`를 실행하여 각 테스트가 독립적인 파서 인스턴스를 갖도록 한다.
- **모킹**: 없음. 외부 의존성 없이 순수 함수/클래스 동작만 테스트한다.

## 테스트 케이스 명세

모든 테스트는 `describe('ExpressionParser', ...)` 블록 안에 위치한다.

---

### `'parses literal strings unchanged'`
- **검증 동작**: `parser.parse('hello world')`는 `['hello world']`를 반환한다. 보간이 없는 순수 리터럴 문자열은 단일 원소 배열로 그대로 반환되어야 한다.
- **사용 메서드**: `parse`

---

### `'parses simple interpolation'`
- **검증 동작**: `parser.parse('hello ${foo}')`는 `['hello ', {path: 'foo'}]`를 반환한다. 텍스트와 경로 바인딩이 혼합된 기본 보간 케이스.
- **사용 메서드**: `parse`

---

### `'parses number interpolation'`
- **검증 동작**: `parser.parse('number is ${num}')`은 `['number is ', {path: 'num'}]`을 반환한다. 숫자 관련 이름의 경로도 동일하게 `{ path: ... }` 형태로 파싱된다.
- **사용 메서드**: `parse`

---

### `'parses nested interpolation'`
- **검증 동작**: `parser.parse('val is ${${nested}}')`는 `['val is ', {path: 'nested'}]`를 반환한다. 중첩된 `${...}` 보간에서 내부 `${nested}`가 재귀적으로 파싱되어 `{path: 'nested'}`로 해소된다.
- **사용 메서드**: `parse`

---

### `'handles escaped interpolation'`
- **검증 동작**: `parser.parse('escaped \\${foo}')`는 `['escaped ', '${', 'foo}']`를 반환한다. 역슬래시로 이스케이프된 `$\{`는 보간으로 처리되지 않고 `'${'` 리터럴 문자열이 삽입되며, 이후 `'foo}'`가 별도 문자열로 분리된다.
- **사용 메서드**: `parse`

---

### `'parses function calls'`
- **검증 동작**: `parser.parse('sum is ${add(a: 10, b: 20)}')`는 `['sum is ', {call: 'add', args: {a: 10, b: 20}, returnType: 'any'}]`를 반환한다. 숫자 인수를 가진 함수 호출이 올바르게 파싱된다.
- **사용 메서드**: `parse`

---

### `'parses function calls with string literals'`
- **검증 동작**: `parser.parse('case is ${upper(text: "hello")}')`는 `['case is ', {call: 'upper', args: {text: 'hello'}, returnType: 'any'}]`를 반환한다. 문자열 리터럴 인수가 따옴표 없이 순수 문자열 값으로 파싱된다.
- **사용 메서드**: `parse`

---

### `'parses keywords'`
- **검증 동작**: `parser.parse('${true} ${false} ${null}')`는 `[true, ' ', false, ' ']`를 반환한다. `true`는 불리언 `true`, `false`는 불리언 `false`, `null`은 빈 문자열 `''`로 변환되어 필터링 후 제거된다.
- **사용 메서드**: `parse`

---

### `'returns error on max depth exceeded'`
- **검증 동작**: `parser.parse('depth', 11)`이 `/Max recursion depth reached/` 패턴과 일치하는 에러를 던진다. `depth` 인수가 `MAX_DEPTH(10)`를 초과하면 즉시 에러가 발생한다.
- **사용 메서드**: `parse` (두 번째 인수 `depth = 11` 직접 전달)

---

### `'handles deep recursion gracefully'`
- **검증 동작**: `parser.parse('${${"hello"}}')`는 `['hello']`를 반환한다. 중첩 보간 안에 문자열 리터럴이 있는 경우 최종적으로 해소된 값을 반환한다.
- **사용 메서드**: `parse`

---

### `'returns error on unclosed interpolation'`
- **검증 동작**: `parser.parse('hello ${world')`가 `/Unclosed interpolation/` 패턴과 일치하는 에러를 던진다. 닫는 `}`가 없는 보간은 에러로 처리된다.
- **사용 메서드**: `parse`

---

### `'returns error on invalid function syntax'`
- **검증 동작**: `parser.parse('${add(a: 1, b: 2}')`가 `/Expected '\)'/` 패턴과 일치하는 에러를 던진다. 함수 호출에서 닫는 `)`가 없으면 에러가 발생한다.
- **사용 메서드**: `parse`

---

### `'returns error on unexpected characters at end'`
- **검증 동작**: `parser.parse('${true false}')`가 `/Unexpected characters/` 패턴과 일치하는 에러를 던진다. 유효한 표현식 뒤에 예상치 못한 추가 문자가 있으면 에러가 발생한다.
- **사용 메서드**: `parse`

---

### `'handles empty identifiers'`
- **검증 동작**: 세 가지를 검증한다:
  1. `parser.parse('${()}')`는 `[{call: '', args: {}, returnType: 'any'}]`를 반환한다. 빈 함수 이름도 허용된다.
  2. `parser.parseExpression('')`는 `''`를 반환한다. 빈 문자열 표현식은 빈 문자열을 반환한다.
  3. `parser.parseExpression('()')`는 `{call: '', args: {}, returnType: 'any'}`를 반환한다. 이름 없는 빈 인수 함수 호출도 파싱된다.
- **사용 메서드**: `parse`, `parseExpression`

---

### `'handles string literals with escaped characters'`
- **검증 동작**: `parser.parseExpression("'line1\\nline2\\t\\r\\'\\\\x'")`는 `"line1\nline2\t\r'\\x"`를 반환한다. `\n`, `\t`, `\r`, `\'`, `\\` 이스케이프 시퀀스가 모두 올바르게 변환된다.
- **사용 메서드**: `parseExpression`

---

### `'handles parsing paths with special characters'`
- **검증 동작**: `parser.parseExpression('my-path.with_underscores')`는 `{path: 'my-path.with_underscores'}`를 반환한다. 경로에 하이픈(`-`)과 밑줄(`_`), 점(`.`)이 포함되어도 하나의 경로로 파싱된다.
- **사용 메서드**: `parseExpression`

---

### `'returns error on missing colon in function args'`
- **검증 동작**: `parser.parseExpression('add(a 10, b: 20)')`가 `/Expected ':'/` 패턴과 일치하는 에러를 던진다. 함수 인수 이름 뒤에 `:`가 없으면 에러가 발생한다.
- **사용 메서드**: `parseExpression`

## 동작 흐름

테스트 파일은 `describe` 블록 하나로 구성된다. `beforeEach`가 각 `it` 블록 실행 전에 `parser` 변수를 새 인스턴스로 초기화한다. 각 `it` 블록은 독립적으로 실행되며, `assert.deepStrictEqual`로 정상 케이스의 반환값을 검증하고, `assert.throws`로 에러 케이스에서 적절한 에러 메시지 패턴을 검증한다.
