# renderers/web_core/src/v0_9/basic_catalog/functions/basic_functions.test.ts

## 개요

이 파일은 `basic_functions.ts`에 정의된 `BASIC_FUNCTIONS` 구현체 전체를 대상으로 하는 통합 테스트 모음이다. Node.js 내장 `node:test` 프레임워크와 `node:assert`를 사용하며, `@preact/signals-core`의 `effect`를 활용해 반응형 시그널 출력 검증도 포함한다.

## 의존성

### 외부 패키지
- `node:test` — `describe`, `it`
- `node:assert` — `assert.strictEqual`, `assert.throws`, `assert.ok`
- `@preact/signals-core` — `effect`, `Signal`

### 저장소 내부 모듈
- [`./basic_functions.ts`](./basic_functions.ts.md) — `BASIC_FUNCTIONS`
- [`../../state/data-model.ts`](../../state/data-model.ts.md) — `DataModel`
- [`../../rendering/data-context.ts`](../../rendering/data-context.ts.md) — `DataContext`
- [`../../errors.ts`](../../errors.ts.md) — `A2uiExpressionError`
- [`../../catalog/types.ts`](../../catalog/types.ts.md) — `Catalog`, `ComponentApi`

## Exports

이 파일은 테스트 파일로 외부로 내보내는 항목이 없다.

## 픽스처 및 공통 설정

### `testCatalog`
`new Catalog<ComponentApi>('test', [], BASIC_FUNCTIONS)` — 테스트 전용 카탈로그. 컴포넌트 없이 `BASIC_FUNCTIONS`만 등록.

### `invoke(name, args, context)`
- **시그니처**: `(name: string, args: Record<string, any>, context: DataContext) => unknown`
- `testCatalog.invoker(name, args, context)` 호출을 감싸는 헬퍼. 모든 산술·비교·논리·문자열·검증 테스트에서 사용.

### `createTestDataContext(model, path, functionInvoker?, locale?)`
- **시그니처**: `(model: DataModel, path: string, functionInvoker?: any, locale?: string) => DataContext`
- `mockSurface` 객체(`dataModel`, `catalog: { invoker }`, `dispatchError: () => {}`, `locale`)를 `any`로 캐스팅해 `new DataContext(mockSurface, path)` 생성.
- `functionInvoker` 기본값은 `testCatalog.invoker`, `locale`은 `undefined`(환경 기본값 사용).

### 공유 픽스처
- `dataModel = new DataModel({ a: 10, b: 20 })` — 산술·비교·포매팅 등 대부분의 테스트에서 공유. `formatString` 데이터 바인딩 테스트에서 `dataModel.set('/a', 42)`로 값을 변경.
- `context = createTestDataContext(dataModel, '/')` — 공유 기본 컨텍스트.

---

## 테스트 케이스 명세

### `describe('Arithmetic')`

#### `it('add')`
- 검증: `{a:1, b:2}` → `3` (정수 합산)
- 검증: `{a:'1', b:'2'}` → `3` (문자열 숫자 강제 변환)
- 검증: `{a:10, b:undefined}` → `A2uiExpressionError` throw
- 검증: `{a:10}` (b 누락) → `A2uiExpressionError` throw

#### `it('subtract')`
- 검증: `{a:5, b:3}` → `2`
- 검증: `{a:10, b:undefined}` 및 b 누락 → `A2uiExpressionError`

#### `it('multiply')`
- 검증: `{a:4, b:2}` → `8`
- 검증: `{a:10, b:undefined}` 및 b 누락 → `A2uiExpressionError`

#### `it('divide')`
- 검증: `{a:10, b:2}` → `5`
- 검증: `{a:10, b:0}` → `Infinity` (0 나눗셈)
- 검증: `{a:10, b:undefined}`, `{a:undefined, b:10}`, `{a:undefined, b:undefined}`, `{a:10, b:null}`, `{a:10, b:'invalid'}` → 모두 `A2uiExpressionError`
- 검증: `{a:10, b:'2'}` → `5`, `{a:'10', b:'2'}` → `5` (문자열 숫자 강제 변환 허용)

---

### `describe('Comparison')`

#### `it('equals')`
- 검증: `{a:1, b:1}` → `true`, `{a:1, b:2}` → `false`
- 검증: a 또는 b 누락 → `A2uiExpressionError`

#### `it('not_equals')`
- 검증: `{a:1, b:2}` → `true`
- 검증: a 또는 b 누락 → `A2uiExpressionError`

#### `it('greater_than')`
- 검증: `{a:5, b:3}` → `true`, `{a:3, b:5}` → `false`
- 검증: `{a:10, b:undefined}`, `{a:10, b:null}`, `{a:10, b:'invalid'}`, a 또는 b 누락 → `A2uiExpressionError`

#### `it('less_than')`
- 검증: `{a:3, b:5}` → `true`
- 검증: `{a:3, b:undefined}`, `{a:3, b:null}`, `{a:3, b:'invalid'}`, a 또는 b 누락 → `A2uiExpressionError`

---

### `describe('Logical')`

#### `it('and')`
- 검증: `{values:[true,true]}` → `true`, `{values:[true,false]}` → `false`
- 검증: 단일 원소 배열 `{values:[true]}` 및 빈 args → `A2uiExpressionError`

#### `it('or')`
- 검증: `{values:[false,true]}` → `true`, `{values:[false,false]}` → `false`
- 검증: 단일 원소 배열 및 빈 args → `A2uiExpressionError`

#### `it('not')`
- 검증: `{value:false}` → `true`, `{value:true}` → `false`
- 검증: 빈 args → `A2uiExpressionError`

---

### `describe('String')`

#### `it('contains')`
- 검증: `{string:'hello world', substring:'world'}` → `true`
- 검증: `{string:'hello world', substring:'foo'}` → `false`
- 검증: `string` 또는 `substring` 누락 → `A2uiExpressionError`

#### `it('starts_with')`
- 검증: `{string:'hello', prefix:'he'}` → `true`
- 검증: `string` 또는 `prefix` 누락 → `A2uiExpressionError`

#### `it('ends_with')`
- 검증: `{string:'hello', suffix:'lo'}` → `true`
- 검증: `string` 또는 `suffix` 누락 → `A2uiExpressionError`

---

### `describe('Validation')`

#### `it('required')`
- 검증: `{value:'a'}` → `true`, `{value:''}` → `false`, `{value:null}` → `false`
- 검증: 빈 args → `A2uiExpressionError`

#### `it('length')`
- 검증: `{value:'abc', min:2}` → `true`, `{value:'abc', max:2}` → `false`
- 검증: 빈 args → `A2uiExpressionError`

#### `it('numeric')`
- 검증: `{value:10, min:5, max:15}` → `true`, `{value:3, min:5}` → `false`
- 검증: 빈 args → `A2uiExpressionError`

#### `it('email')`
- 유효 케이스(→ `true`): `'test@example.com'`, `'test.name@example.com'`, `'test+label@example.com'`, `'test@example-domain.com'`
- 무효 케이스(→ `false`): `'invalid'`, `'test@test'`, `'test@test.c'`, `'test@.com'`
- 검증: 빈 args → `A2uiExpressionError`

#### `it('regex')`
- 검증: `{value:'abc', pattern:'^[a-z]+$'}` → `true`
- 검증: `{value:'123', pattern:'^[a-z]+$'}` → `false`

#### `it('regex handles invalid pattern')`
- 검증: 잘못된 패턴 `'['` → `A2uiExpressionError` throw (정규식 생성 실패)

---

### `describe('Formatting')`

포매팅 테스트는 시그널 기반 비동기 검증을 위해 `effect` + `done` 콜백 패턴을 사용한다. `cleanup` 변수에 `effect()` 반환값을 저장해 검증 완료 후 구독을 해제한다.

#### `it('formatString (static literal)', _, done)`
- 픽스처: 공유 `context`
- 검증: `{value:'hello world'}` 호출 결과가 `Signal<string>`이며, `effect` 내에서 `.value === 'hello world'`

#### `it('formatString (with data binding)', _, done)`
- 픽스처: `dataModel = {a:10, b:20}`
- 검증 1 (`emitCount=0`): `{value:'Value: ${a}'}` → `'Value: 10'`
- 검증 2 (`emitCount=1`): `setTimeout`으로 `dataModel.set('/a', 42)` 실행 후 시그널 재계산 → `'Value: 42'` (반응형 업데이트 확인)

#### `it('formatString (with function call)', _, done)`
- 픽스처: `functionInvoker`를 `add` 함수만 처리하는 커스텀 함수로 교체한 `ctxWithInvoker`
- 검증: `{value:'Result: ${add(a: 5, b: 7)}'}` → `'Result: 12'`

#### `it('formatString (object value is JSON-stringified)', _, done)`
- 픽스처: `new DataModel({user:{name:'Alice', age:30}})`
- 검증: `{value:'User: ${user}'}` → `'User: {"name":"Alice","age":30}'` (객체의 JSON 직렬화)

#### `it('formatString (array value is JSON-stringified)', _, done)`
- 픽스처: `new DataModel({tags:['swift','ios']})`
- 검증: `{value:'Tags: ${tags}'}` → `'Tags: ["swift","ios"]'`

#### `it('formatString (nested array is JSON-stringified)', _, done)`
- 픽스처: `new DataModel({matrix:[[1,2],[3,4]]})`
- 검증: `{value:'M = ${matrix}'}` → `'M = [[1,2],[3,4]]'`

#### `it('formatString (array with null is JSON-stringified preserving nulls)', _, done)`
- 픽스처: `new DataModel({vals:[1,null,3]})`
- 검증: `{value:'V = ${vals}'}` → `'V = [1,null,3]'` (JSON.stringify에서 null이 보존됨)

#### `it('formatString (null/undefined interpolated as empty string)', _, done)`
- 픽스처: `new DataModel({x:null})`
- 검증: `{value:'val=${x}end'}` → `'val=end'` (null은 빈 문자열로 변환)

#### `it('formatNumber')`
- 검증: `{value:1234.56, decimals:1}` 결과가 `string`이며, `'1,234.6'`, `'1234.6'`, `'1 234,6'` 중 하나를 포함 (`Intl` 동작이 환경에 따라 다를 수 있어 유연하게 검증)

#### `it('formatCurrency')`
- 검증: `{value:1234.56, currency:'USD'}` 결과가 `string`이며, 금액(`'1,234.56'` 또는 `'1234.56'`)과 통화 기호(`'$'` 또는 `'USD'`)를 포함

#### `it('formatDate')`
- 검증: `{value:'2025-01-01T12:00:00Z', format:'yyyy-MM-dd'}` → `'2025-01-01'`
- 검증: `{value:'2025-01-01T12:00:00Z', format:'ISO'}` → `'2025-01-01T12:00:00.000Z'`

#### `it('formatDate handles invalid dates')`
- 검증: `{value:'invalid-date', format:'yyyy'}` → `''` (잘못된 날짜는 빈 문자열)

#### `it('formatCurrency fallback on formatting error')`
- 검증: `{value:1234.56, currency:'INVALID-CURRENCY', decimals:2}` → `'1234.56'` (`toFixed(2)` 폴백)

#### `it('pluralize')`
- 검증: `{value:1, one:'apple', other:'apples'}` → `'apple'`
- 검증: `{value:2, one:'apple', other:'apples'}` → `'apples'`

#### `it('pluralize with Welsh locale')`
- 픽스처: `createTestDataContext(dataModel, '/', testCatalog.invoker, 'cy')` (Welsh 로케일)
- args: `{zero:'cathod', one:'gath', two:'gath', few:'cath', many:'chath', other:'cath'}`
- 검증: 값 0 → `'cathod'`, 1 → `'gath'`, 2 → `'gath'`, 3 → `'cath'`, 6 → `'chath'`, 4 → `'cath'` (Welsh의 6가지 복수형 케이스 모두 검증)

#### `it('pluralize fallback to other')`
- 검증: `{value:5, one:'apple', other:'apples'}` → `'apples'` (`many`/`other` 규칙)
- 검증: `{value:1, other:'apples'}` (one 없음) → `'apples'` (`other` 폴백)
- 검증: `{value:0, other:'apples'}` (zero 없음) → `'apples'` (`other` 폴백)

---

### `describe('Actions')`

#### `it('openUrl')`
- 모킹: `global.window`를 `{ open: (url: string) => { openedUrl = url } }` 객체로 교체, `try/finally`로 원래 `window` 복원
- 검증: `{url:'https://google.com'}` 호출 후 `openedUrl === 'https://google.com'`
- 검증: 빈 args `{}` → `A2uiExpressionError`

## 동작 흐름

테스트 파일 최상위에서 `testCatalog`와 공유 `dataModel`·`context`를 초기화한 후, 각 `describe` 블록이 카테고리별로 `invoke` 헬퍼를 통해 함수를 실행한다. 동기 함수는 `assert.strictEqual`·`assert.throws`로 즉시 검증하고, `FormatStringImplementation`처럼 시그널을 반환하는 비동기 함수는 `effect` + `done` 패턴으로 신호 값을 구독하여 검증한 뒤 `cleanup()`으로 구독을 해제한다.
