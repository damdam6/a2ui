# renderers/web_core/src/v0_9/state/surface-group-model.ts

## 개요

A2UI 시스템의 루트 상태 모델로, 활성화된 모든 Surface의 컬렉션을 관리한다. Surface 추가·제거 시 라이프사이클 이벤트를 발행하며, 소속된 어느 Surface에서 액션이 발생하더라도 이를 단일 `onAction` 스트림으로 집약하여 상위 계층에 전달한다.

## 의존성

### 저장소 내부 모듈

- [`./surface-model.ts`](./surface-model.ts.md) — `SurfaceModel`
- [`../catalog/types.ts`](../catalog/types.ts.md) — `ComponentApi`
- [`../common/events.ts`](../common/events.ts.md) — `EventEmitter`, `EventSource`, `Subscription`
- [`../schema/client-to-server.ts`](../schema/client-to-server.ts.md) — `A2uiClientAction`

### 외부 패키지

없음

## Exports

| 이름 | 종류 |
|---|---|
| `SurfaceGroupModel<T>` | 클래스 (제네릭) |

## 상세 명세

### 클래스 `SurfaceGroupModel<T extends ComponentApi>`

#### 필드

| 필드 | 타입 | 접근 | 설명 |
|---|---|---|---|
| `surfaces` | `Map<string, SurfaceModel<T>>` | `private` | surfaceId → SurfaceModel 매핑 |
| `surfaceUnsubscribers` | `Map<string, Subscription>` | `private` | surfaceId → onAction 구독 해제 핸들 |
| `_onSurfaceCreated` | `EventEmitter<SurfaceModel<T>>` | `private readonly` | Surface 추가 이벤트 발행기 |
| `_onSurfaceDeleted` | `EventEmitter<string>` | `private readonly` | Surface 제거 이벤트 발행기 |
| `_onAction` | `EventEmitter<A2uiClientAction>` | `private readonly` | 전체 Surface 액션 집약 발행기 |
| `onSurfaceCreated` | `EventSource<SurfaceModel<T>>` | `readonly` | 외부 구독 전용 |
| `onSurfaceDeleted` | `EventSource<string>` | `readonly` | 외부 구독 전용 |
| `onAction` | `EventSource<A2uiClientAction>` | `readonly` | 외부 구독 전용 |

#### `addSurface(surface: SurfaceModel<T>): void`

1. `surfaces.has(surface.id)`이면 `console.warn`을 출력하고 즉시 반환한다(에러 없이 무시).
2. `surfaces.set(surface.id, surface)`.
3. `surface.onAction.subscribe(action => this._onAction.emit(action))`으로 해당 Surface의 액션을 그룹 레벨로 전파하는 구독을 등록하고, 반환된 `Subscription`을 `surfaceUnsubscribers.set(surface.id, sub)`에 저장한다.
4. `_onSurfaceCreated.emit(surface)`.

#### `deleteSurface(id: string): void`

1. `surfaces.get(id)`로 Surface를 찾는다. 없으면 아무 작업도 하지 않는다.
2. `surfaceUnsubscribers.get(id)`로 구독 해제 핸들을 찾아 `sub.unsubscribe()`를 호출한 뒤 `surfaceUnsubscribers.delete(id)`.
3. `surfaces.delete(id)`.
4. `surface.dispose()`.
5. `_onSurfaceDeleted.emit(id)`.

#### `getSurface(id: string): SurfaceModel<T> | undefined`

`surfaces.get(id)`를 그대로 반환한다.

#### getter `surfacesMap`

시그니처: `get surfacesMap(): ReadonlyMap<string, SurfaceModel<T>>`

`surfaces` Map을 `ReadonlyMap`으로 노출한다. 외부에서 직접 수정 불가.

#### `dispose(): void`

1. `Array.from(this.surfaces.keys())`로 ID 목록을 스냅샷한 뒤 각 ID에 대해 `deleteSurface(id)`를 호출한다. (`deleteSurface` 내부에서 Map을 수정하므로 이터레이션 중 수정을 피하기 위해 배열로 변환).
2. `_onSurfaceCreated.dispose()`, `_onSurfaceDeleted.dispose()`, `_onAction.dispose()` 순으로 이벤트 발행기들을 정리한다.

## 동작 흐름

`SurfaceGroupModel`은 A2UI 세션의 최상위 상태 컨테이너다. 서버가 새 Surface를 생성하거나 삭제할 때 `addSurface`/`deleteSurface`가 호출된다. 각 Surface의 `onAction`은 자동으로 그룹의 `onAction`으로 전파되므로, 외부 전송 계층은 그룹의 `onAction` 하나만 구독하면 모든 Surface의 액션을 수신할 수 있다. `dispose()` 호출 시 소속된 모든 Surface가 연쇄적으로 정리된다.
