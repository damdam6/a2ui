# renderers/web_core/src/v0_9/rendering/generic-binder.test.ts

## 개요

`GenericBinder`의 `CHECKABLE` 트레잇(validation 배열 처리) 동작을 검증하는 단위 테스트 파일이다. Zod 스키마로 정의된 `checks` 프로퍼티가 반응형으로 검증 상태(`isValid`, `validationErrors`)를 산출하는지, 복수 규칙 집계, 기본 메시지 처리, 빈 배열 처리 등을 테스트한다. Node.js 내장 `node:test`와 `node:assert`를 사용한다.

## 의존성

### 외부 패키지
- `node:assert` — 단언 함수
- `node:test` — `describe`, `it`
- `zod` — `z` (mock 함수 스키마 및 컴포넌트 스키마 정의)

### 저장소 내부 모듈
- [`./generic-binder.js`](./generic-binder.ts.md) — `GenericBinder` (테스트 대상)
- [`./component-context.js`](./component-context.ts.md) — `ComponentContext`
- [`../state/surface-model.js`](../state/surface-model.ts.md) — `SurfaceModel`
- [`../catalog/types.js`](../catalog/types.ts.md) — `Catalog`
- [`../state/component-model.js`](../state/component-model.ts.md) — `ComponentModel`
- [`../schema/common-types.js`](../schema/common-types.ts.md) — `CommonSchemas`

## Exports

없음 (테스트 파일)

## 픽스처 및 헬퍼

### `mockCatalog`

`new Catalog('test', [], [])`로 생성된 공유 빈 카탈로그. 모든 테스트에서 재사용된다.

### `setupSurfaceAndMocks()`

`SurfaceModel`을 생성하고 아래 두 함수를 `surface.catalog.functions` 맵에 직접 주입하는 팩토리 함수다. 반환값은 `{surface, schema}`.

주입하는 함수:
- `'required'`: `(args) => !!args.value` — 값의 truthy 여부를 반환한다. 스키마: `z.object({value: z.any()})`.
- `'min_length'`: `(args) => typeof args.value === 'string' && args.value.length >= args.min` — 문자열 최소 길이를 검사한다. 스키마: `z.object({value: z.any(), min: z.number()})`.

`surface.catalog.invoker`는 위 함수 맵을 조회하는 클로저로 교체된다.

`schema`는 다음 Zod 객체:
```
z.object({
  value: CommonSchemas.DynamicString,
  checks: CommonSchemas.Checkable.shape.checks,
})
```

## 테스트 케이스 상세

### `describe('GenericBinder Checkable Trait')`

#### `should resolve checkable validation state reactively`

- **설정**: `ComponentModel('c1', 'Test', {...})`에 `value: {path: '/val'}`, `checks: [{condition: {call: 'required', args: {value: {path: '/val'}}}, message: 'Value is required'}]` 설정. `/val`을 `''`로 초기화.
- **검증 (초기 상태)**: `binder.snapshot.isValid === false`, `binder.snapshot.validationErrors === ['Value is required']`.
- **변경**: `surface.dataModel.set('/val', 'hello')` 후 비동기 대기(`setTimeout(0)`).
- **검증 (변경 후)**: `binder.snapshot.isValid === true`, `binder.snapshot.validationErrors === []`.
- **사용 픽스처**: `setupSurfaceAndMocks`, `ComponentModel('c1', ...)`, `ComponentContext(surface, 'c1')`, `GenericBinder.subscribe(() => {})`.

#### `should aggregate multiple validation rules correctly`

- **설정**: `ComponentModel('c2', 'Test', {...})`에 `required`와 `min_length(3)` 두 개의 체크 규칙 적용. `/val`을 `''`로 초기화.
- **단계별 검증**:
  1. 초기: `isValid === false`, `validationErrors === ['Cannot be empty', 'Must be at least 3 characters']`.
  2. `/val = 'hi'` 변경 후: `isValid === false`, `validationErrors === ['Must be at least 3 characters']`.
  3. `/val = 'hello'` 변경 후: `isValid === true`, `validationErrors === []`.
- **사용 픽스처**: `setupSurfaceAndMocks`, `ComponentModel('c2', ...)`, `ComponentContext`, `GenericBinder.subscribe`.

#### `should provide a default message if rule.message is missing`

- **설정**: `ComponentModel('c3', 'Test', {...})`에 `message` 프로퍼티 없는 체크 규칙 (`as any` 캐스팅). `/val`은 `''`.
- **검증**: `binder.snapshot.isValid === false`, `binder.snapshot.validationErrors === ['Validation failed']` (기본 메시지 `'Validation failed'`가 삽입된다).
- **주의**: `subscribe` 없이 `snapshot`을 직접 읽는다 — `GenericBinder` 생성자에서 초기 해석이 동기적으로 실행됨을 검증한다.

#### `should default to valid if checks array is empty or undefined`

- **설정**: `ComponentModel('c4', 'Test', { value: 'static', checks: [] })`으로 빈 체크 배열 사용.
- **검증**: `binder.snapshot.isValid === true`, `binder.snapshot.validationErrors === []`.
- **주의**: `subscribe` 없이 `snapshot`을 직접 읽어 초기 동기 상태를 검증한다.

## 동작 흐름

1. 각 테스트마다 `setupSurfaceAndMocks()`로 격리된 `SurfaceModel`과 Zod 스키마를 준비한다.
2. `ComponentModel`을 생성하고 `surface.componentsModel.addComponent`로 등록한다.
3. `ComponentContext(surface, componentId)`로 컴포넌트 컨텍스트를 만들고 `GenericBinder(context, schema)`로 바인더를 초기화한다.
4. 반응형 테스트는 `binder.subscribe(() => {})`를 호출하여 연결(connect) 상태를 활성화한 뒤, `surface.dataModel.set`으로 데이터를 변경하고 `setTimeout(0)`으로 마이크로태스크 완료를 기다린 후 `binder.snapshot`을 단언한다.
5. 정적 검증 테스트(`c3`, `c4`)는 `subscribe` 없이 `binder.snapshot`을 직접 읽어 생성자 내 초기 동기 해석 결과를 검증한다.
