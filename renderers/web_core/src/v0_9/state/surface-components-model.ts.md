# renderers/web_core/src/v0_9/state/surface-components-model.ts

## 개요

특정 서피스(surface)에 속한 UI 컴포넌트들의 컬렉션을 관리하는 모델 클래스다. 컴포넌트의 추가·제거를 담당하며, 각 이벤트(생성/삭제)를 구독자에게 전달한다. 중복 ID 추가 시 에러를 던지고, 제거 시에는 컴포넌트의 `dispose()`를 자동 호출한다.

## 의존성

### 저장소 내부 모듈

- [`./component-model.ts`](./component-model.ts.md) — `ComponentModel` 임포트
- [`../common/events.ts`](../common/events.ts.md) — `EventEmitter`, `EventSource` 임포트
- [`../errors.ts`](../errors.ts.md) — `A2uiStateError` 임포트

### 외부 패키지

없음

## Exports

- `SurfaceComponentsModel` (클래스)

## 상세 명세

### `SurfaceComponentsModel` 클래스

#### 필드

| 필드 | 종류 | 타입 | 설명 |
|---|---|---|---|
| `components` | `private` | `Map<string, ComponentModel>` | 컴포넌트 ID → 인스턴스 매핑. 삽입 순서를 유지한다. |
| `_onCreated` | `private readonly` | `EventEmitter<ComponentModel>` | 컴포넌트 추가 시 이벤트를 발생시키는 내부 에미터. |
| `_onDeleted` | `private readonly` | `EventEmitter<string>` | 컴포넌트 삭제 시 ID를 발생시키는 내부 에미터. |
| `onCreated` | `readonly` 공개 | `EventSource<ComponentModel>` | 구독 전용 생성 이벤트 소스. |
| `onDeleted` | `readonly` 공개 | `EventSource<string>` | 구독 전용 삭제 이벤트 소스. |

---

#### `get(id: string): ComponentModel | undefined`

`components.get(id)`로 해당 ID의 컴포넌트를 반환한다. 없으면 `undefined`.

---

#### `get entries(): IterableIterator<[string, ComponentModel]>`

`components.entries()`를 반환한다. 삽입 순서대로 `[id, ComponentModel]` 쌍을 순회할 수 있다.

---

#### `addComponent(component: ComponentModel): void`

1. `components.has(component.id)`가 참이면 `A2uiStateError('Component with id \'{id}\' already exists.')` 던짐.
2. `components.set(component.id, component)`으로 등록.
3. `_onCreated.emit(component)`으로 생성 이벤트 발생.

---

#### `removeComponent(id: string): void`

1. `components.get(id)`로 컴포넌트를 조회한다. 존재하지 않으면 아무것도 하지 않는다(에러 없음).
2. 존재하면: `components.delete(id)`, `component.dispose()`, `_onDeleted.emit(id)` 순서로 실행.

---

#### `dispose(): void`

1. `components` 의 모든 값에 대해 `component.dispose()` 호출.
2. `components.clear()`로 맵을 비운다.
3. `_onCreated.dispose()`, `_onDeleted.dispose()`로 이벤트 에미터 정리.

## 동작 흐름

1. `addComponent()`로 컴포넌트를 등록하면 내부 Map에 저장되고 `onCreated` 이벤트가 발생한다.
2. `removeComponent()`는 Map에서 제거한 뒤 컴포넌트 자체의 `dispose()`를 호출하고 `onDeleted`에 ID를 전달한다.
3. `dispose()`는 모든 컴포넌트를 순차적으로 해제하고 내부 상태와 이벤트 에미터를 모두 정리한다.
