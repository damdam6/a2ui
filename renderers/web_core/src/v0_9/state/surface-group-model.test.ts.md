# renderers/web_core/src/v0_9/state/surface-group-model.test.ts

## 개요

`SurfaceGroupModel`의 Surface 추가·삭제·이벤트 알림, 액션 전파, 중복 처리, `dispose` 동작을 검증하는 단위 테스트 모음이다.

## 의존성

### 저장소 내부 모듈

- [`./surface-group-model.ts`](./surface-group-model.ts.md) — `SurfaceGroupModel`
- [`../catalog/types.ts`](../catalog/types.ts.md) — `Catalog`, `ComponentApi`
- [`./surface-model.ts`](./surface-model.ts.md) — `SurfaceModel`

### 외부 패키지

- `node:assert`
- `node:test` — `describe`, `it`, `beforeEach`

## Exports

없음 (테스트 파일)

## 테스트 케이스 명세

### 픽스처

`beforeEach`에서 다음을 생성한다:
- `model = new SurfaceGroupModel<ComponentApi>()`
- `catalog = new Catalog('test-catalog', [])`

### 케이스 목록

| 케이스명 | 검증 동작 | 픽스처/모킹 |
|---|---|---|
| `adds surface` | `addSurface` 후 `getSurface('s1')`이 동일 인스턴스를 반환하는지 확인 | `new SurfaceModel('s1', catalog, {})` |
| `ignores duplicate surface addition` | 동일 ID로 두 번 `addSurface` 시 기존 Surface가 유지되는지 확인 (에러 없음) | `s1`, `s2` 모두 ID `'s1'` |
| `deletes surface` | `addSurface` 후 `deleteSurface` 호출 시 `getSurface` → `undefined` | `new SurfaceModel('s1', catalog, {})` |
| `notifies lifecycle listeners` | `onSurfaceCreated`/`onSurfaceDeleted` 구독 후 추가·삭제 시 올바른 인스턴스와 ID가 콜백에 전달되는지 확인 | `created`, `deletedId` 변수 |
| `propagates actions from surfaces` | `onAction` 구독 후 `surface.dispatchAction`을 `await` 호출하면 `receivedAction`에 `name`, `surfaceId`, `sourceComponentId`가 올바르게 포함되는지 확인 | `receivedAction` 변수, `async` 테스트 |
| `stops propagating actions after deletion` | `addSurface` → `deleteSurface` 후 `dispatchAction` 호출 시 `onAction` 콜백이 호출되지 않음 확인 | `callCount` 카운터, `async` 테스트 |
| `exposes surfacesMap` | Surface 추가 후 `surfacesMap.size === 1` 및 올바른 인스턴스 참조 확인 | `model.surfacesMap` |
| `disposes correctly` | 두 Surface를 추가하고 `onSurfaceDeleted` 구독 후 `dispose()` 호출 시 `deletedCount === 2`이고 `surfacesMap.size === 0` 확인 | `deletedCount` 카운터 |

## 동작 흐름

각 테스트는 독립적인 `SurfaceGroupModel`과 빈 `Catalog`에서 시작한다. 액션 전파 테스트(`propagates actions`, `stops propagating`)는 `async`/`await`를 사용한다. `surfacesMap` 테스트는 getter의 `ReadonlyMap` 반환을 검증한다. `disposes correctly`는 `dispose()` 한 번으로 두 Surface 모두에 대해 `onSurfaceDeleted`가 발행됨을 카운터로 확인한다.
