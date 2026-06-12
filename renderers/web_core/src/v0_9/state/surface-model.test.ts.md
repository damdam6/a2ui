# renderers/web_core/src/v0_9/state/surface-model.test.ts

## 개요

`SurfaceModel`의 초기화, 데이터 모델 접근, 액션 디스패치, 에러 디스패치, `ComponentContext` 생성, `dispose` 동작을 검증하는 단위 테스트 모음이다.

## 의존성

### 저장소 내부 모듈

- [`./surface-model.ts`](./surface-model.ts.md) — `SurfaceModel`
- [`../catalog/types.ts`](../catalog/types.ts.md) — `Catalog`, `ComponentApi`
- [`./component-model.ts`](./component-model.ts.md) — `ComponentModel`
- [`../rendering/component-context.ts`](../rendering/component-context.ts.md) — `ComponentContext`

### 외부 패키지

- `node:assert`
- `node:test` — `describe`, `it`, `beforeEach`

## Exports

없음 (테스트 파일)

## 테스트 케이스 명세

### 픽스처

`beforeEach`에서 다음을 생성한다:
- `actions = []`, `errors = []` 배열 초기화
- `catalog = new Catalog('test-catalog', [])`
- `surface = new SurfaceModel<ComponentApi>('surface-1', catalog, {})`
- `surface.onAction.subscribe(async action => actions.push(action))`
- `surface.onError.subscribe(async error => errors.push(error))`

### 케이스 목록

| 케이스명 | 검증 동작 | 픽스처/모킹 |
|---|---|---|
| `initializes with empty data model` | `surface.dataModel.get('/')` → `{}` | 없음 |
| `exposes components model` | `componentsModel.addComponent` 후 `componentsModel.get('c1')` truthy | `new ComponentModel('c1', 'Button', {})` |
| `dispatches actions with metadata` | `dispatchAction({event: {name: 'click', context: {foo: 'bar'}}}, 'comp-1')` 후 `actions[0]`에 `name`, `surfaceId`, `sourceComponentId`, `context`, `timestamp`가 올바르게 포함되는지 확인. `timestamp`는 유효한 Date 문자열인지 `new Date()`로 검증 | `await`, `actions` 배열 |
| `dispatches actions with default context` | `context` 없이 `dispatchAction` 호출 시 `context` → `{}` | `actions` 배열 |
| `dispatches errors` | `dispatchError({code: 'TEST_ERROR', message: 'Something failed'})` 후 `errors[0]`에 `code`, `message`, `surfaceId` 포함 확인 | `errors` 배열 |
| `creates a component context` | `ComponentContext` 인스턴스 생성 후 `ctx.dataContext.path === '/mydata'` 확인 | `new ComponentModel('root', 'Box', {})`, `new ComponentContext(surface, 'root', '/mydata')` |
| `disposes resources` | `dispose()` 후 `dispatchAction` 호출 시 `onAction` 콜백이 호출되지 않음 확인 (`EventEmitter.dispose`가 모든 리스너를 지웠기 때문) | `actionReceived` 플래그 |

## 동작 흐름

각 테스트는 독립적인 `SurfaceModel` 인스턴스에서 시작하며, `onAction`/`onError` 구독은 `beforeEach`에서 미리 등록된다. 액션/에러 관련 테스트는 `async`/`await`를 사용한다. `dispose` 테스트는 `EventEmitter.dispose()`가 호출된 이후 발행된 이벤트가 구독자에게 도달하지 않음을 확인하며, 이때 `dispatchAction`의 반환 Promise는 무시된다(결과를 `await`하지 않음).
