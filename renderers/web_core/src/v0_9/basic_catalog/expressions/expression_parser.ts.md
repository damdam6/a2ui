# renderers/web_core/src/v0_9/basic_catalog/expressions/expression_parser.ts

## 개요

A2UI 표현식 문자열을 파싱하여 `DynamicValue` 배열로 변환하는 파서 모듈이다. `${...}` 플레이스홀더를 포함한 문자열을 리터럴(문자열·숫자·불리언), 경로 기반 데이터 바인딩(`{ path: ... }`), 이름 있는 인수를 가진 함수 호출(`{ call: ..., args: ..., returnType: 'any' }`)으로 분해한다. 파서는 재귀 깊이 제한을 두어 스택 오버플로를 방지하고, 파싱 실패 시 `A2uiExpressionError`를 던진다.

## 의존성

### 외부 패키지
없음.

### 저장소 내부 모듈
- [`../../schema/common-types.ts`](../../schema/common-types.ts.md) — `DynamicValue` 타입 임포트.
- [`../../errors.ts`](../../errors.ts.md) — `A2uiExpressionError` 클래스 임포트.

## Exports

| 이름 | 종류 | 설명 |
|---|---|---|
| `ExpressionParser` | `class` | 표현식 파싱의 진입점이 되는 공개 파서 클래스 |

`Scanner` 클래스는 파일 내부에만 존재하며 export되지 않는다.

## 상세 명세

---

### class `ExpressionParser`

표현식 파싱 전반을 담당하는 공개 클래스. 인스턴스 상태를 가지지 않아 재사용 가능하다.

#### 정적 필드

- `private static readonly MAX_DEPTH: number = 10` — 중첩 표현식의 최대 재귀 깊이. 초과 시 `A2uiExpressionError`를 던진다.

#### 공개 메서드

##### `parse(input: string, depth = 0): DynamicValue[]`

혼합 문자열(리터럴 텍스트 + `${...}` 보간)을 `DynamicValue` 배열로 변환한다.

동작 단계:
1. `depth > MAX_DEPTH`이면 즉시 `A2uiExpressionError('Max recursion depth reached in parse')` 던짐.
2. `input`이 비어 있거나 `'${'`를 포함하지 않으면 `[input]`을 그대로 반환.
3. `Scanner(input)`을 생성하고 루프를 시작한다.
4. 루프 내에서 세 가지 경우를 구분한다:
   - `'${'`로 시작하면: 2글자 전진, `extractInterpolationContent`로 내용 추출, `parseExpression(content, depth + 1)` 재귀 호출, 결과가 `null`이 아니면 `parts`에 추가.
   - `\$\{` (이스케이프된 보간)이면: 1글자 전진(백슬래시 소비), `'${'` 문자열 리터럴을 `parts`에 추가, 추가로 2글자 전진.
   - 그 외: 다음 `'${'` 또는 `\$\{`가 나타날 때까지 스캔하여 substring을 `parts`에 추가.
5. 루프 후 `parts`에서 `null`과 빈 문자열을 필터링하여 반환.

##### `parseExpression(expr: string, depth = 0): DynamicValue`

단일 표현식 문자열을 하나의 `DynamicValue`로 파싱한다. `${...}` 내부에서 추출된 내용이 입력된다.

동작 단계:
1. `expr`을 `trim()`한다. 빈 문자열이면 `''`를 반환.
2. `Scanner(expr)`을 생성하고 `parseExpressionInternal`을 호출한다.
3. 호출 후 스캐너가 끝에 도달하지 않았으면(`!scanner.isAtEnd()`) `A2uiExpressionError('Unexpected characters at end of expression: ...')` 던짐.
4. 결과를 반환.

#### 비공개 메서드

##### `private extractInterpolationContent(scanner: Scanner): string`

`'${'` 직후부터 대응하는 `'}'`까지의 내용을 추출한다.

동작:
- `braceBalance = 1`로 시작하여 `{`를 만나면 증가, `}`를 만나면 감소한다.
- `'` 또는 `"`를 만나면 문자열 리터럴 내부로 진입하여 이스케이프(`\`)를 처리하며 닫는 따옴표까지 소비한다.
- `braceBalance`가 0이 되면 종료. 종료 전까지 스캐너가 끝에 도달하면(`braceBalance > 0`) `A2uiExpressionError("Unclosed interpolation: missing '}'")` 던짐.
- 반환값: `start` 위치부터 현재 위치 − 1(닫는 `}`는 제외)까지의 substring.

##### `private parseExpressionInternal(scanner: Scanner, depth: number): DynamicValue`

실제 표현식 분기 파싱 로직.

단계:
1. 공백 스킵. 스캐너 끝이면 `''` 반환.
2. **중첩 보간**: `'${'`로 시작하면 2글자 전진, `extractInterpolationContent`, `parseExpression(content, depth + 1)` 재귀 호출.
3. **문자열 리터럴**: 현재 문자가 `'` 또는 `"`이면 `parseStringLiteral` 호출.
4. **숫자 리터럴**: 현재 문자가 숫자(`0`–`9`)이면 `parseNumberLiteral` 호출.
5. **키워드**: `'true'`이면 `true` 반환, `'false'`이면 `false` 반환, `'null'`이면 `''` 반환. (`matchesKeyword`는 알파벳/숫자/밑줄이 뒤따르지 않을 때만 매칭.)
6. **식별자/경로**: `scanPathOrIdentifier`로 토큰 스캔, 공백 스킵 후:
   - 다음 문자가 `'('`이면 `parseFunctionCall(token, scanner, depth)` 호출.
   - 그렇지 않으면 `{ path: token }` 반환. `token`이 빈 문자열이면 `''` 반환.

##### `private scanPathOrIdentifier(scanner: Scanner): string`

경로 또는 식별자를 스캔한다. 허용 문자: 영숫자(`isAlnum`), `'/'`, `'.'`, `'_'`, `'-'`. 허용되지 않는 문자를 만나면 중단. 스캔된 substring 반환.

##### `private parseFunctionCall(funcName: string, scanner: Scanner, depth: number): { call: string; args: Record<string, any>; returnType: 'any' }`

함수 호출 표현식 `name(argName: value, ...)` 파싱.

단계:
1. `'('`를 소비(`scanner.match('(')`). 공백 스킵.
2. `')'`가 아닌 동안 루프:
   a. `scanIdentifier`로 인수 이름 스캔.
   b. 공백 스킵 후 `':'` 소비. 실패하면 `A2uiExpressionError("Expected ':' ...")` 던짐.
   c. 공백 스킵 후 `parseExpressionInternal`로 인수 값 파싱.
   d. 공백 스킵 후 `','`이 있으면 소비 및 공백 스킵.
3. `')'` 소비. 실패하면 `A2uiExpressionError("Expected ')' ...")` 던짐.
4. `{ call: funcName, args, returnType: 'any' }` 반환.

##### `private scanIdentifier(scanner: Scanner): string`

순수 식별자(`isAlnum` 또는 `'_'` 문자만) 스캔. 첫 비허용 문자에서 중단. substring 반환.

##### `private parseStringLiteral(scanner: Scanner): string`

단일(`'`) 또는 이중(`"`) 따옴표로 감싼 문자열 리터럴 파싱. 이스케이프 처리: `\n` → 줄바꿈, `\t` → 탭, `\r` → 캐리지 리턴, `\\` → `\`, 나머지 `\x` → `x` (단순 다음 문자 그대로). 닫는 따옴표에서 중단. 결과 문자열 반환.

##### `private parseNumberLiteral(scanner: Scanner): number`

숫자 및 `'.'` 문자로만 구성된 토큰을 스캔하여 `Number(...)` 변환 후 반환. 정수 및 부동소수점 모두 처리.

##### `private isAlnum(c: string): boolean`

`c`가 `'a'`–`'z'`, `'A'`–`'Z'`, `'0'`–`'9'` 범위인지 확인. 범위 비교(`>=`, `<=`)로 구현.

##### `private isDigit(c: string): boolean`

`c`가 `'0'`–`'9'` 범위인지 확인.

---

### class `Scanner` (비공개, 파일 내부 전용)

문자 단위 스캐닝을 위한 헬퍼 클래스.

#### 필드
- `pos: number = 0` — 현재 스캔 위치(인덱스).
- `input: string` — 생성자 인수로 받은 원본 문자열.

#### 메서드

| 시그니처 | 동작 |
|---|---|
| `constructor(input: string)` | `this.input = input` 설정. `pos = 0`. |
| `isAtEnd(): boolean` | `pos >= input.length` |
| `peek(offset = 0): string` | `pos + offset` 위치 문자 반환. 범위 초과 시 `'\0'`. |
| `advance(count = 1): string` | `pos`부터 `count`글자 substring 반환 후 `pos += count`. |
| `match(expected: string): boolean` | 현재 문자가 `expected`와 같으면 1글자 전진 후 `true`, 아니면 `false`. |
| `matches(expected: string): boolean` | `input.startsWith(expected, pos)` — 전진하지 않음. |
| `matchesString(expected: string): boolean` | `peek() === expected` — 단일 문자 비교, 전진하지 않음. |
| `matchesKeyword(keyword: string): boolean` | `keyword`로 시작하고 그 다음 문자가 `[a-zA-Z0-9_]`가 아닐 때만 `keyword.length`만큼 전진 후 `true`. |
| `skipWhitespace()` | `\s` 정규식에 매칭되는 문자들을 연속으로 전진하여 건너뜀. |

## 동작 흐름

외부 진입점은 `ExpressionParser`의 `parse` 또는 `parseExpression` 두 메서드다. `parse`는 혼합 문자열 전체를 처리하며, `'${'`를 찾을 때마다 `extractInterpolationContent`로 중괄호 균형을 추적하여 내용을 분리한 뒤 `parseExpression`에 위임한다. `parseExpression`은 단일 표현식을 받아 내부적으로 `parseExpressionInternal`로 분기 파싱을 수행한다. 중첩 `${...}` 표현식은 재귀 호출로 처리되며, 매 재귀마다 `depth`를 증가시켜 `MAX_DEPTH(10)` 초과 시 에러를 발생시킨다. 모든 문자 소비는 `Scanner` 인스턴스를 통해 이루어지며, `Scanner`는 순수하게 위치 추적과 문자 접근만 담당한다.
