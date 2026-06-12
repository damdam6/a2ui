# renderers/web_core/src/v0_9/basic_catalog/functions/basic_functions.ts

## 개요

이 파일은 a2ui Basic Catalog에서 사용되는 표준 함수 구현체들을 정의한다. 산술(Arithmetic), 비교(Comparison), 논리(Logical), 문자열(String), 유효성 검증(Validation), 포매팅(Formatting), 액션(Actions) 등 7개 카테고리에 걸쳐 총 25개의 함수 구현체를 제공하며, 이를 `BASIC_FUNCTIONS` 배열로 한데 묶어 내보낸다. 각 구현체는 `createFunctionImplementation`으로 생성되어 API 스펙과 실행 로직이 결합된 `FunctionImplementation` 객체로 만들어진다.

## 의존성

### 외부 패키지
- `@preact/signals-core` — `computed`, `isSignal` 활용 (반응형 시그널 처리)
- `date-fns` — `format` 함수 (날짜 포매팅)

### 저장소 내부 모듈
- [`../expressions/expression_parser.ts`](../expressions/expression_parser.ts.md) — `ExpressionParser` (템플릿 문자열 파싱)
- [`../../catalog/types.ts`](../../catalog/types.ts.md) — `createFunctionImplementation`, `FunctionImplementation`, `isSignal`
- [`./basic_functions_api.ts`](./basic_functions_api.ts.md) — 각 함수의 API 스펙 (매개변수 타입 정의): `AddApi`, `SubtractApi`, `MultiplyApi`, `DivideApi`, `EqualsApi`, `NotEqualsApi`, `GreaterThanApi`, `LessThanApi`, `AndApi`, `OrApi`, `NotApi`, `ContainsApi`, `StartsWithApi`, `EndsWithApi`, `RequiredApi`, `RegexApi`, `LengthApi`, `NumericApi`, `EmailApi`, `FormatStringApi`, `FormatNumberApi`, `FormatCurrencyApi`, `FormatDateApi`, `PluralizeApi`, `OpenUrlApi`
- [`../../errors.ts`](../../errors.ts.md) — `A2uiExpressionError`

## Exports

| 이름 | 종류 | 설명 |
|------|------|------|
| `AddImplementation` | 상수 (`FunctionImplementation`) | 덧셈 |
| `SubtractImplementation` | 상수 (`FunctionImplementation`) | 뺄셈 |
| `MultiplyImplementation` | 상수 (`FunctionImplementation`) | 곱셈 |
| `DivideImplementation` | 상수 (`FunctionImplementation`) | 나눗셈 |
| `EqualsImplementation` | 상수 (`FunctionImplementation`) | 동등 비교 |
| `NotEqualsImplementation` | 상수 (`FunctionImplementation`) | 불일치 비교 |
| `GreaterThanImplementation` | 상수 (`FunctionImplementation`) | 초과 비교 |
| `LessThanImplementation` | 상수 (`FunctionImplementation`) | 미만 비교 |
| `AndImplementation` | 상수 (`FunctionImplementation`) | 논리 AND |
| `OrImplementation` | 상수 (`FunctionImplementation`) | 논리 OR |
| `NotImplementation` | 상수 (`FunctionImplementation`) | 논리 NOT |
| `ContainsImplementation` | 상수 (`FunctionImplementation`) | 문자열 포함 여부 |
| `StartsWithImplementation` | 상수 (`FunctionImplementation`) | 문자열 시작 여부 |
| `EndsWithImplementation` | 상수 (`FunctionImplementation`) | 문자열 끝 여부 |
| `RequiredImplementation` | 상수 (`FunctionImplementation`) | 필수 값 검증 |
| `RegexImplementation` | 상수 (`FunctionImplementation`) | 정규식 검증 |
| `LengthImplementation` | 상수 (`FunctionImplementation`) | 길이 범위 검증 |
| `NumericImplementation` | 상수 (`FunctionImplementation`) | 숫자 범위 검증 |
| `EmailImplementation` | 상수 (`FunctionImplementation`) | 이메일 형식 검증 |
| `FormatStringImplementation` | 상수 (`FunctionImplementation`) | 템플릿 문자열 포매팅 |
| `FormatNumberImplementation` | 상수 (`FunctionImplementation`) | 숫자 포매팅 |
| `FormatCurrencyImplementation` | 상수 (`FunctionImplementation`) | 통화 포매팅 |
| `FormatDateImplementation` | 상수 (`FunctionImplementation`) | 날짜 포매팅 |
| `PluralizeImplementation` | 상수 (`FunctionImplementation`) | 복수형 선택 |
| `OpenUrlImplementation` | 상수 (`FunctionImplementation`) | URL 열기 |
| `BASIC_FUNCTIONS` | 상수 (`FunctionImplementation[]`) | 위 25개 구현체의 배열 |

## 상세 명세

### 모듈 수준 캐시

세 개의 `Map` 캐시가 모듈 수준에서 선언된다.

- `numberFormatCache: Map<string, Intl.NumberFormat>` — 숫자 포매터 캐시. 키 형식: `"${locale ?? 'default'}:${decimals ?? 'undef'}:${grouping ?? 'true'}"`
- `currencyFormatCache: Map<string, Intl.NumberFormat>` — 통화 포매터 캐시. 키 형식: `"${locale ?? 'default'}:${currency}:${decimals ?? 'undef'}:${grouping ?? 'true'}"`
- `pluralRulesCache: Map<string, Intl.PluralRules>` — 복수형 규칙 캐시. 키 형식: `locale ?? 'default'`

---

### 산술 구현체

#### `AddImplementation`
- **API**: `AddApi`
- **로직**: `args.a + args.b` — 두 수의 합을 반환. 타입 강제는 API 스펙에서 처리.

#### `SubtractImplementation`
- **API**: `SubtractApi`
- **로직**: `args.a - args.b` — `b`를 `a`에서 뺀 값을 반환.

#### `MultiplyImplementation`
- **API**: `MultiplyApi`
- **로직**: `args.a * args.b` — 두 수의 곱을 반환.

#### `DivideImplementation`
- **API**: `DivideApi`
- **로직**:
  1. `args.a` 또는 `args.b`가 `undefined` 또는 `null`이면 `NaN`을 반환.
  2. `Number(a)` 또는 `Number(b)`가 `NaN`이면 `NaN`을 반환.
  3. `numB === 0`이면 `Infinity`를 반환.
  4. 그 외에는 `numA / numB`를 반환.

---

### 비교 구현체

#### `EqualsImplementation`
- **API**: `EqualsApi`
- **로직**: `args.a === args.b` — 엄격한 동등 비교.

#### `NotEqualsImplementation`
- **API**: `NotEqualsApi`
- **로직**: `args.a !== args.b` — 엄격한 불일치 비교.

#### `GreaterThanImplementation`
- **API**: `GreaterThanApi`
- **로직**: `args.a > args.b`

#### `LessThanImplementation`
- **API**: `LessThanApi`
- **로직**: `args.a < args.b`

---

### 논리 구현체

#### `AndImplementation`
- **API**: `AndApi`
- **로직**: `args.values` 배열의 모든 요소를 `Boolean` 강제 변환한 후, `Array.prototype.every`로 모두 truthy인지 확인. 하나라도 falsy면 `false` 반환.

#### `OrImplementation`
- **API**: `OrApi`
- **로직**: `args.values` 배열에서 `Array.prototype.some`으로 하나라도 truthy인 요소가 있으면 `true` 반환.

#### `NotImplementation`
- **API**: `NotApi`
- **로직**: `!args.value` — 단일 값의 논리 부정.

---

### 문자열 구현체

#### `ContainsImplementation`
- **API**: `ContainsApi`
- **로직**: `args.string.includes(args.substring)` — 문자열 내 부분 문자열 포함 여부.

#### `StartsWithImplementation`
- **API**: `StartsWithApi`
- **로직**: `args.string.startsWith(args.prefix)`

#### `EndsWithImplementation`
- **API**: `EndsWithApi`
- **로직**: `args.string.endsWith(args.suffix)`

---

### 유효성 검증 구현체

#### `RequiredImplementation`
- **API**: `RequiredApi`
- **로직**:
  1. `null` 또는 `undefined`이면 `false` 반환.
  2. 타입이 `string`이고 빈 문자열(`''`)이면 `false` 반환.
  3. 배열이고 `length === 0`이면 `false` 반환.
  4. 그 외 `true` 반환.

#### `RegexImplementation`
- **API**: `RegexApi`
- **로직**: `new RegExp(args.pattern).test(args.value)`로 패턴 매칭. `RegExp` 생성 시 예외가 발생하면 `A2uiExpressionError`를 throw하며 원인 오류를 내부에 포함. 에러 메시지 형식: `` `Invalid regex pattern: ${args.pattern}` ``.

#### `LengthImplementation`
- **API**: `LengthApi`
- **로직**:
  1. `args.value`가 `string` 또는 배열이면 `.length`를 `len`으로 사용, 그 외에는 `len = 0`.
  2. `args.min`이 정의되어 있고 NaN이 아닌데 `len < args.min`이면 `false`.
  3. `args.max`가 정의되어 있고 NaN이 아닌데 `len > args.max`이면 `false`.
  4. 그 외 `true` 반환.

#### `NumericImplementation`
- **API**: `NumericApi`
- **로직**:
  1. `isNaN(args.value)`이면 `false`.
  2. `args.min`이 정의되어 있고 NaN이 아닌데 `args.value < args.min`이면 `false`.
  3. `args.max`가 정의되어 있고 NaN이 아닌데 `args.value > args.max`이면 `false`.
  4. 그 외 `true` 반환.

#### `EmailImplementation`
- **API**: `EmailApi`
- **로직**: 정규식 `/^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/`으로 `args.value`를 검사. 주석에 따르면 이는 기본적인 검사이며 모든 이메일 표준을 완전히 준수하지 않는다(TODO: 전용 패키지 사용 권고).

---

### 내부 헬퍼 함수

#### `coerceToString(value: unknown): string`
- **가시성**: 비공개 (파일 내부 전용)
- **로직**:
  1. `null` 또는 `undefined`이면 빈 문자열 `''` 반환.
  2. `typeof value === 'object'`이면 `JSON.stringify(value)` 시도; 실패 시 `String(value)` 반환.
  3. 그 외 `String(value)` 반환.
- a2ui_protocol.md의 "Type conversion" 규칙을 따름.

#### `getNumberFormat(locale: string | undefined, decimals?: number, grouping?: boolean): Intl.NumberFormat`
- **가시성**: 비공개
- **로직**: `numberFormatCache`에서 키로 포매터를 조회. 없으면 `new Intl.NumberFormat(locale, { minimumFractionDigits: decimals, maximumFractionDigits: decimals, useGrouping: grouping })`로 생성 후 캐시에 저장.

#### `getCurrencyFormat(locale: string | undefined, currency: string, decimals?: number, grouping?: boolean): Intl.NumberFormat`
- **가시성**: 비공개
- **로직**: `currencyFormatCache`에서 조회. 없으면 `style: 'currency'`와 `currency` 옵션을 포함한 `Intl.NumberFormat`으로 생성 후 캐시에 저장.

#### `getPluralRules(locale: string | undefined): Intl.PluralRules`
- **가시성**: 비공개
- **로직**: `pluralRulesCache`에서 조회. 없으면 `new Intl.PluralRules(locale)`로 생성 후 캐시에 저장.

---

### 포매팅 구현체

#### `FormatStringImplementation`
- **API**: `FormatStringApi`
- **두 번째 인자**: `context` (실행 컨텍스트, `resolveSignal` 메서드 보유)
- **로직**:
  1. `new ExpressionParser().parse(args.value)`로 템플릿을 파싱해 `parts` 배열을 얻음.
  2. `parts`가 비어 있으면 빈 문자열 `''` 반환.
  3. 각 `part`를 순회: primitive(문자열·숫자·불리언)는 그대로 유지, 객체형(표현식 노드)은 `context.resolveSignal(part)`로 시그널로 변환 → `dynamicParts` 배열 생성.
  4. `computed(() => dynamicParts.map(p => coerceToString(isSignal(p) ? p.value : p)).join(''))` 형태의 반응형 시그널을 반환.
  - 반환값은 `Signal<string>` 타입이며, 참조된 시그널 값이 변경되면 자동으로 재계산됨.

#### `FormatNumberImplementation`
- **API**: `FormatNumberApi`
- **두 번째 인자**: `context` (`.locale` 프로퍼티 사용)
- **로직**:
  1. `isNaN(args.value)`이면 빈 문자열 `''` 반환.
  2. `getNumberFormat(context.locale, args.decimals, args.grouping).format(args.value)` 시도.
  3. 예외 발생 시 `console.warn` 후 폴백: `args.decimals`가 정의되어 있으면 `args.value.toFixed(args.decimals)`, 그 외 `String(args.value)`.

#### `FormatCurrencyImplementation`
- **API**: `FormatCurrencyApi`
- **두 번째 인자**: `context`
- **로직**:
  1. `isNaN(args.value)`이면 빈 문자열 반환.
  2. `getCurrencyFormat(context.locale, args.currency, args.decimals, args.grouping).format(args.value)` 시도.
  3. 예외 발생 시 `console.warn` 후 폴백: `args.value.toFixed(args.decimals ?? 2)`.

#### `FormatDateImplementation`
- **API**: `FormatDateApi`
- **로직**:
  1. `args.value`가 falsy면 빈 문자열 반환.
  2. `new Date(args.value)`로 변환 후 `isNaN(date.getTime())`이면 빈 문자열 반환.
  3. `args.format === 'ISO'`이면 `date.toISOString()` 반환.
  4. 그 외 `date-fns`의 `format(date, args.format)` 시도.
  5. 예외 발생 시 `console.warn` 후 `date.toISOString()` 폴백.

#### `PluralizeImplementation`
- **API**: `PluralizeApi`
- **두 번째 인자**: `context`
- **로직**:
  1. `getPluralRules(context.locale).select(args.value)`로 복수형 규칙 키(`'zero'`|`'one'`|`'two'`|`'few'`|`'many'`|`'other'`) 획득.
  2. `args[rule]`을 `String`으로 변환하여 반환. 해당 키가 없으면 `args.other` 사용, 그것도 없으면 빈 문자열.
  3. 예외 발생 시 `console.warn` 후 `String(args.other ?? '')` 폴백.

---

### 액션 구현체

#### `OpenUrlImplementation`
- **API**: `OpenUrlApi`
- **로직**: `args.url`이 존재하고, `typeof window !== 'undefined'`이며 `window.open`이 있을 때 `window.open(args.url, '_blank')` 호출. 조건 미충족 시 아무 것도 하지 않음(no-op).

---

### `BASIC_FUNCTIONS`
- **타입**: `FunctionImplementation[]`
- **값**: 위 25개 구현체를 선언 순서대로 담은 배열: `[AddImplementation, SubtractImplementation, MultiplyImplementation, DivideImplementation, EqualsImplementation, NotEqualsImplementation, GreaterThanImplementation, LessThanImplementation, AndImplementation, OrImplementation, NotImplementation, ContainsImplementation, StartsWithImplementation, EndsWithImplementation, RequiredImplementation, RegexImplementation, LengthImplementation, NumericImplementation, EmailImplementation, FormatStringImplementation, FormatNumberImplementation, FormatCurrencyImplementation, FormatDateImplementation, PluralizeImplementation, OpenUrlImplementation]`

## 동작 흐름

모든 구현체는 `createFunctionImplementation(Api, handler)` 패턴을 따른다. API 스펙은 `basic_functions_api.ts`에 정의되어 있으며, 호출 시 API가 먼저 인자를 검증·변환한 뒤 handler로 제어가 넘어온다. 포매팅 함수(`FormatNumberImplementation`, `FormatCurrencyImplementation`, `PluralizeImplementation`)는 `Intl` 객체를 반복 생성하는 비용을 줄이기 위해 모듈 수준 캐시를 활용한다. `FormatStringImplementation`만 `@preact/signals-core`의 `computed`를 반환하여 반응형 시그널 체인에 참여한다. 나머지 구현체는 즉시 값을 반환하는 순수 함수 방식으로 동작한다.
