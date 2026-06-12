# renderers/web_core/src/v0_9/rendering/data-context.test.ts

## 개요

`DataContext` 클래스의 동작을 검증하는 단위 테스트 파일이다. 경로 해석(절대/상대), 리터럴 및 함수 호출의 동기·반응형 평가, 액션 해석, 에러 처리(ZodError, A2uiExpressionError, 일반 Error) 등 핵심 시나리오를 망라한다. Node.js 내장 `node:test` 프레임워크와 `node:assert`를 사용한다.

## 의존성

### 외부 패키지
- `node:assert` — 단언(assertion) 함수
- `node:test` — `describe`, `it`, `beforeEach`
- `@preact/signals-core` — `signal`, `computed` (픽스처 생성용)
- `zod` — `z` (ZodError 픽스처 생성용)

### 저장소 내부 모듈
- [`../state/data-model.js`](../state/data-model.ts.md) — `DataModel`
- [`./data-context.js`](./data-context.ts.md) — `DataContext` (테스트 대상)
- [`../errors.js`](../errors.ts.md) — `A2uiExpressionError`

## Exports

없음 (테스트 파일)

## 픽스처 및 헬퍼

### `createTestDataContext(model, path, functionInvoker?, dispatchError?)`

- **매개변수**: `DataModel`, 경로 문자열, 선택적 `functionInvoker` (기본값: `() => null`), 선택적 `dispatchError` 콜백 (기본값: `() => {}`)
- **동작**: `{ dataModel, catalog: { invoker }, dispatchError }`를 가진 `mockSurface` 객체를 `as any`로 캐스팅하여 `new DataContext(mockSurface, path)`를 반환한다. 실제 `SurfaceModel` 없이 `DataContext`를 격리 테스트하기 위한 팩토리 함수다.

### `beforeEach` 픽스처

`DataModel`을 다음 초기 데이터로 생성:
```
{ user: { name: 'Alice', address: { city: 'Wonderland' } }, list: ['a', 'b'] }
```
`context`는 `'/user'` 경로로 생성된다.

## 테스트 케이스 상세

### `describe('DataContext')`

| 테스트명 | 검증 동작 |
|----------|-----------|
| `resolves relative paths` | `{path: 'name'}`을 `'/user'` 컨텍스트에서 해석하면 `'Alice'`를 반환한다. |
| `resolves absolute paths` | `{path: '/list/0'}`은 절대 경로로 처리되어 `'a'`를 반환한다. |
| `resolves nested paths` | `{path: 'address/city'}`가 `'/user/address/city'`로 해석되어 `'Wonderland'`를 반환한다. |
| `updates data via relative path` | `context.set('name', 'Bob')` 호출 후 `model.get('/user/name')`이 `'Bob'`이 된다. |
| `creates nested context` | `context.nested('address')`의 `path`가 `'/user/address'`이고, 그 컨텍스트에서 `{path: 'city'}`를 해석하면 `'Wonderland'`가 반환된다. |
| `handles root context` | `'/'` 경로 컨텍스트에서 `{path: 'user/name'}`을 해석하면 `'Alice'`가 반환된다. |
| `subscribes relative path` | `{path: 'name'}` 구독 후 `context.set('name', 'Charlie')`를 호출하면 `onChange`가 `'Charlie'`로 호출된다. |
| `resolves using resolveDynamicValue() method with literals` | 문자열 리터럴 `'literal'`, `{path: 'name'}`, `{path: '/list/0'}` 세 케이스를 동시에 검증한다. |
| `resolves literal arrays` | 배열 `['literal', 'array']`는 배열 그대로 반환된다. |
| `subscribes literal arrays as static` | 배열 리터럴을 구독할 때 초기 값이 원본 배열이고, 이후 `set` 호출로도 `onChange`가 호출되지 않는다. |
| `resolves function calls synchronously` | `fnInvoker`가 `'add'` 함수를 구현할 때 `{call: 'add', args: {a:1, b:2}}`의 결과가 `3`이다. |
| `dispatches generic error on function call without invoker synchronously` | invoker가 에러를 throw하면 `resolveDynamicValue`가 `undefined`를 반환하고, `dispatchError`가 `code: 'EXPRESSION_ERROR'`인 에러 객체로 호출된다. |
| `does not resolve arbitrary objects recursively` | `{foo: 'bar', nested: {path: 'name'}, list: [...]}` 같은 임의 객체는 재귀 해석 없이 그대로 반환된다. |
| `subscribes to literal objects as signals without resolution` | `{foo: 'bar', nested: {path: 'name'}}` 객체는 리터럴 신호로 감싸지고, 내부 `{path: ...}`는 해석되지 않는다. 이후 경로 변경에도 신호 값이 바뀌지 않는다. |
| `subscribes to function calls with no args` | 인수 없는 함수 호출 구독 시 `onChange`가 즉시 호출되지 않는다. |
| `returns undefined on function call without invoker reactively` | invoker가 에러를 throw하는 반응형 구독에서 초기 값이 `undefined`다. |
| `subscribes to function call returning a signal` | invoker가 `signal('hello')`를 반환하는 경우 크래시 없이 동작하고, 초기 `val`은 `undefined`다(동기 구독 제외). |
| `subscribes to invalid dynamic value reactively (falls back to literal signal)` | `{unknown: 'thing'}` 같이 알 수 없는 객체는 리터럴 신호로 처리되어 초기 값이 원본 객체다. |
| `handles path resolution edge cases` | `nested('')`와 `nested('.')` 모두 현재 경로를 유지한다. 루트 컨텍스트에서 `nested('test')`는 `'/test'`이고, 끝에 `/`가 붙은 경로에서 `nested('test')`는 끝 슬래시를 제거하고 결합한다. |
| `subscribes to function call with arguments reactively` | `{call: 'greet', args: {name: {path: 'name'}}}` 구독 시 초기 값이 `'Hello Alice'`이고, `name`을 `'Bob'`으로 변경하면 즉시 `'Hello Bob'`으로 갱신된다. `unsubscribe` 호출로 정리된다. |

### `describe('resolveAction')`

| 테스트명 | 검증 동작 |
|----------|-----------|
| `resolves event actions non-recursively` | `action.event.context`의 1단계 `DynamicValue`만 해석한다. 예시에서 `id: {path: 'name'}`은 `'Alice'`로, `metadata: {nested: {path: 'something'}}`은 그대로(비재귀) 유지된다. |
| `resolves functionCall actions` | `{functionCall: {call: 'greet', args: {name: {path: 'name'}}}}`을 `resolveAction`하면 `'Hello Alice'`를 반환한다. |

### `describe('Error Handling')`

| 테스트명 | 검증 동작 | 픽스처/모킹 |
|----------|-----------|-------------|
| `translates ZodError into A2uiExpressionError and dispatches error` | ZodError를 throw하는 invoker 사용 시 `result === undefined`, `dispatchedError.code === 'EXPRESSION_ERROR'`, `dispatchedError.expression === 'fail'`이 성립한다. | `z.ZodError([{code: 'invalid_type', ...}])`를 throw하는 mock invoker |
| `dispatches generic Error as EXPRESSION_ERROR to surface` | 일반 Error를 throw하는 invoker 사용 시 `code`, `expression`, `message`, `details.stack`이 모두 올바르게 설정된다. | `message: 'Generic failure'`, `stack: 'Mock stack trace'`를 가진 Error를 throw하는 mock invoker |
| `dispatches A2uiExpressionError to surface` | `A2uiExpressionError`를 throw하는 invoker 사용 시 `expression`이 `'custom_func'`로 설정된다. | `new A2uiExpressionError('Custom expr error', 'custom_func')`를 throw하는 mock invoker |
| `handles errors thrown during reactive argument resolution` | `A2uiExpressionError`를 발생시키는 `computed` 신호를 반환하는 내부 함수 사용 시, 트리거가 `true`가 되면 구독 값이 `undefined`가 되고 `dispatchedError.message === 'Inner failure'`가 성립한다. | `signal(false)` 트리거, `computed`로 조건부 에러를 발생시키는 mock invoker |
| `handles generic errors thrown during reactive execution` | 위와 동일하나 일반 Error를 사용하며, `dispatchedError.code === 'EXPRESSION_ERROR'`, `dispatchedError.message === 'Generic inner failure'`가 성립한다. | `signal(false)` 트리거, `computed`로 조건부 일반 Error를 발생시키는 mock invoker |

## 동작 흐름

각 테스트는 `createTestDataContext` 헬퍼로 실제 `SurfaceModel` 없이 격리된 `DataContext`를 만든다. `beforeEach`에서 공통 `DataModel`과 `'/user'` 경로 컨텍스트를 초기화한다. 일부 테스트는 별도의 `fnInvoker`나 `dispatchError` 콜백을 직접 주입하여 함수 호출과 에러 처리 경로를 검증한다. 반응형 테스트는 `context.set()`으로 데이터를 변경하고 콜백 또는 구독 값이 갱신되는지 확인한다.
