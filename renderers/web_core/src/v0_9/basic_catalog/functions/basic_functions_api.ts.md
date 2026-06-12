# renderers/web_core/src/v0_9/basic_catalog/functions/basic_functions_api.ts

## 개요

A2UI 기본 카탈로그에서 제공하는 모든 내장 함수의 API 명세(디스크립터)를 정의하는 파일이다. 각 디스크립터 상수는 함수 이름(`name`), 반환 타입(`returnType`), 그리고 Zod 스키마(`schema`)를 담고 있으며, 런타임 인수 검증에 사용된다. 실제 실행 로직(implementation)은 포함하지 않고 API 계약만 선언한다. 파일 끝의 `BASIC_FUNCTION_APIS` 배열이 모든 디스크립터를 집합으로 제공한다.

## 의존성

### 외부 패키지
- `zod` — 런타임 스키마 검증 및 타입 추론

### 저장소 내부 모듈
없음

## Exports

| 이름 | 종류 | 설명 |
|---|---|---|
| `AddApi` | 상수(객체) | 덧셈 함수 API 디스크립터 |
| `SubtractApi` | 상수(객체) | 뺄셈 함수 API 디스크립터 |
| `MultiplyApi` | 상수(객체) | 곱셈 함수 API 디스크립터 |
| `DivideApi` | 상수(객체) | 나눗셈 함수 API 디스크립터 |
| `EqualsApi` | 상수(객체) | 동등 비교 함수 API 디스크립터 |
| `NotEqualsApi` | 상수(객체) | 비동등 비교 함수 API 디스크립터 |
| `GreaterThanApi` | 상수(객체) | 초과 비교 함수 API 디스크립터 |
| `LessThanApi` | 상수(객체) | 미만 비교 함수 API 디스크립터 |
| `AndApi` | 상수(객체) | 논리 AND 함수 API 디스크립터 |
| `OrApi` | 상수(객체) | 논리 OR 함수 API 디스크립터 |
| `NotApi` | 상수(객체) | 논리 NOT 함수 API 디스크립터 |
| `ContainsApi` | 상수(객체) | 문자열 포함 여부 함수 API 디스크립터 |
| `StartsWithApi` | 상수(객체) | 문자열 접두사 확인 함수 API 디스크립터 |
| `EndsWithApi` | 상수(객체) | 문자열 접미사 확인 함수 API 디스크립터 |
| `RequiredApi` | 상수(객체) | 필수값 검증 함수 API 디스크립터 |
| `RegexApi` | 상수(객체) | 정규식 검증 함수 API 디스크립터 |
| `LengthApi` | 상수(객체) | 문자열 길이 검증 함수 API 디스크립터 |
| `NumericApi` | 상수(객체) | 숫자 범위 검증 함수 API 디스크립터 |
| `EmailApi` | 상수(객체) | 이메일 형식 검증 함수 API 디스크립터 |
| `FormatStringApi` | 상수(객체) | 문자열 보간(interpolation) 함수 API 디스크립터 |
| `FormatNumberApi` | 상수(객체) | 숫자 포맷 함수 API 디스크립터 |
| `FormatCurrencyApi` | 상수(객체) | 통화 포맷 함수 API 디스크립터 |
| `FormatDateApi` | 상수(객체) | 날짜 포맷 함수 API 디스크립터 |
| `PluralizeApi` | 상수(객체) | 복수형 변환 함수 API 디스크립터 |
| `OpenUrlApi` | 상수(객체) | URL 열기 함수 API 디스크립터 |
| `BASIC_FUNCTION_APIS` | 상수(배열) | 위 모든 API 디스크립터를 담은 집합 배열 |

## 상세 명세

각 API 디스크립터 객체는 동일한 구조 `{ name, returnType, schema }` 를 따른다.

### 공통 패턴

숫자형 인수에는 `z.preprocess(v => (v === null ? undefined : v), z.coerce.number())` 패턴을 사용한다. `null` 입력을 `undefined`로 변환한 뒤 강제 형 변환(`coerce`)한다. 문자열 인수에는 `z.preprocess(v => (v === undefined ? undefined : String(v)), z.string())` 패턴이 사용되며, `undefined`가 아닌 모든 값을 문자열로 변환한다. 임의 타입 인수에는 `z.any().refine(v => v !== undefined, 'Required')` 패턴으로 `undefined`만 거부한다.

---

### 산술 연산

#### `AddApi`
- `name`: `'add'`
- `returnType`: `'number'`
- 스키마: `a`(숫자형 coerce), `b`(숫자형 coerce) — 두 수의 합

#### `SubtractApi`
- `name`: `'subtract'`
- `returnType`: `'number'`
- 스키마: `a`(숫자형 coerce), `b`(숫자형 coerce) — `a`에서 `b`를 뺀 값

#### `MultiplyApi`
- `name`: `'multiply'`
- `returnType`: `'number'`
- 스키마: `a`(숫자형 coerce), `b`(숫자형 coerce) — 두 수의 곱

#### `DivideApi`
- `name`: `'divide'`
- `returnType`: `'number'`
- 스키마: `a`(숫자형 coerce, 피제수), `b`(숫자형 coerce, 제수)

---

### 비교 연산

#### `EqualsApi`
- `name`: `'equals'`
- `returnType`: `'boolean'`
- 스키마: `a`(any, undefined 불가), `b`(any, undefined 불가) — 두 값의 동등 여부

#### `NotEqualsApi`
- `name`: `'not_equals'`
- `returnType`: `'boolean'`
- 스키마: `a`(any, undefined 불가), `b`(any, undefined 불가) — 두 값의 비동등 여부

#### `GreaterThanApi`
- `name`: `'greater_than'`
- `returnType`: `'boolean'`
- 스키마: `a`(숫자형 coerce), `b`(숫자형 coerce) — `a > b`

#### `LessThanApi`
- `name`: `'less_than'`
- `returnType`: `'boolean'`
- 스키마: `a`(숫자형 coerce), `b`(숫자형 coerce) — `a < b`

---

### 논리 연산

#### `AndApi`
- `name`: `'and'`
- `returnType`: `'boolean'`
- 스키마: `values` — any 배열, 최소 2개 필수. 배열 내 모든 요소의 논리 AND

#### `OrApi`
- `name`: `'or'`
- `returnType`: `'boolean'`
- 스키마: `values` — any 배열, 최소 2개 필수. 배열 내 하나라도 true면 true

#### `NotApi`
- `name`: `'not'`
- `returnType`: `'boolean'`
- 스키마: `value`(any, undefined 불가) — 단일 값의 논리 부정

---

### 문자열 연산

#### `ContainsApi`
- `name`: `'contains'`
- `returnType`: `'boolean'`
- 스키마: `string`(String 변환), `substring`(String 변환) — `string`이 `substring`을 포함하는지 여부

#### `StartsWithApi`
- `name`: `'starts_with'`
- `returnType`: `'boolean'`
- 스키마: `string`(String 변환), `prefix`(String 변환) — `string`이 `prefix`로 시작하는지 여부

#### `EndsWithApi`
- `name`: `'ends_with'`
- `returnType`: `'boolean'`
- 스키마: `string`(String 변환), `suffix`(String 변환) — `string`이 `suffix`로 끝나는지 여부

---

### 유효성 검사

#### `RequiredApi`
- `name`: `'required'`
- `returnType`: `'boolean'`
- 스키마: `value`(any, undefined 불가) — 값이 null/undefined/빈 값이 아닌지 검사

#### `RegexApi`
- `name`: `'regex'`
- `returnType`: `'boolean'`
- 스키마: `value`(String 변환), `pattern`(String 변환) — `value`가 `pattern` 정규식과 일치하는지 여부

#### `LengthApi`
- `name`: `'length'`
- `returnType`: `'boolean'`
- 스키마: `value`(any, undefined 불가), `min`(숫자 optional), `max`(숫자 optional). `min`과 `max` 중 적어도 하나는 반드시 제공해야 하며, 미제공 시 `"Must provide either 'min' or 'max'"` 오류 발생

#### `NumericApi`
- `name`: `'numeric'`
- `returnType`: `'boolean'`
- 스키마: `value`(숫자 coerce), `min`(숫자 optional), `max`(숫자 optional). `LengthApi`와 동일하게 `min`/`max` 중 하나 이상 필수

#### `EmailApi`
- `name`: `'email'`
- `returnType`: `'boolean'`
- 스키마: `value`(String 변환) — 유효한 이메일 주소 형식 여부

---

### 포맷 변환

#### `FormatStringApi`
- `name`: `'formatString'`
- `returnType`: `'any'`
- 스키마: `value`(문자열 coerce) — `${expression}` 구문으로 모델 경로 및 함수를 보간한다. JSON Pointer 절대/상대 경로 `${/path}`, 함수 호출 `${funcName(arg:value)}`, 리터럴 `${` 이스케이프 `\\${` 지원

#### `FormatNumberApi`
- `name`: `'formatNumber'`
- `returnType`: `'string'`
- 스키마: `value`(숫자 coerce), `decimals`(숫자 optional), `grouping`(boolean, 기본값 `true`) — 천단위 구분자 및 소수점 자릿수를 지정해 숫자를 문자열로 변환

#### `FormatCurrencyApi`
- `name`: `'formatCurrency'`
- `returnType`: `'string'`
- 스키마: `value`(숫자 coerce), `currency`(문자열 coerce, 예: `"USD"`), `decimals`(숫자 optional), `grouping`(boolean, 기본값 `true`) — 통화 코드 포함 통화 문자열로 변환

#### `FormatDateApi`
- `name`: `'formatDate'`
- `returnType`: `'string'`
- 스키마: `value`(any, undefined 불가), `format`(문자열 coerce) — Unicode TR35 날짜 패턴 토큰 사용. 지원 토큰: `yy`/`yyyy`(연도), `M`/`MM`/`MMM`/`MMMM`(월), `d`/`dd`/`E`/`EEEE`(일), `h`/`hh`(12시간), `H`/`HH`(24시간), `mm`(분), `ss`(초), `a`(AM/PM)

#### `PluralizeApi`
- `name`: `'pluralize'`
- `returnType`: `'string'`
- 스키마: `value`(숫자 coerce), `zero`(문자열 optional), `one`(문자열 optional), `two`(문자열 optional), `few`(문자열 optional), `many`(문자열 optional), `other`(문자열, 필수) — CLDR 복수 카테고리에 따라 적절한 문자열 반환. `other`은 항상 폴백으로 필수

---

### 액션

#### `OpenUrlApi`
- `name`: `'openUrl'`
- `returnType`: `'void'`
- 스키마: `url`(String 변환) — 지정된 URL을 브라우저 또는 핸들러에서 열기. 반환값 없음

---

### `BASIC_FUNCTION_APIS`

위 25개 API 디스크립터를 선언 순서대로 담은 배열 상수: `AddApi`, `SubtractApi`, `MultiplyApi`, `DivideApi`, `EqualsApi`, `NotEqualsApi`, `GreaterThanApi`, `LessThanApi`, `AndApi`, `OrApi`, `NotApi`, `ContainsApi`, `StartsWithApi`, `EndsWithApi`, `RequiredApi`, `RegexApi`, `LengthApi`, `NumericApi`, `EmailApi`, `FormatStringApi`, `FormatNumberApi`, `FormatCurrencyApi`, `FormatDateApi`, `PluralizeApi`, `OpenUrlApi`.

## 동작 흐름

이 파일은 순수하게 선언만 담고 있으며, 런타임 부작용은 없다. 임포트한 쪽에서 `BASIC_FUNCTION_APIS` 배열을 순회해 각 디스크립터를 `Catalog`에 등록하거나, 개별 디스크립터를 `createFunctionImplementation`에 전달해 실제 구현과 결합한다. Zod 스키마는 `Catalog.invoker`가 `fn.schema.parse(rawArgs)` 호출 시 입력 인수를 검증·강제 변환하는 데 사용된다.
