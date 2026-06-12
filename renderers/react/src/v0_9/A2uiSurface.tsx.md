# renderers/react/src/v0_9/A2uiSurface.tsx

## 개요

A2UI v0_9 React 렌더러의 루트 렌더링 엔트리포인트를 제공하는 파일이다. `SurfaceModel`을 받아 컴포넌트 트리를 React로 렌더링하는 `A2uiSurface`, 컴포넌트 존재 여부를 구독하여 지연 렌더링을 처리하는 `DeferredChild`, 실제 컴포넌트 구현체를 렌더링하는 `ResolvedChild` 세 컴포넌트로 구성된다. `useSyncExternalStore`를 활용하여 외부 모델의 변경(생성/삭제 이벤트)에 반응적으로 동작한다.

## 의존성

### 외부 패키지

- `react` — `React`, `useSyncExternalStore`, `memo`, `useMemo`, `useCallback`

### 저장소 내부 모듈

- `@a2ui/web_core/v0_9` — `SurfaceModel`, `ComponentContext`, `ComponentModel` 타입
- [`./adapter`](./adapter.tsx.md) — `ReactComponentImplementation` 타입

## Exports

| 이름 | 종류 | 설명 |
|------|------|------|
| `DeferredChild` | `React.FC` (memo) | 컴포넌트 ID로 조회하여 로드될 때까지 대기하는 래퍼 |
| `A2uiSurface` | `React.FC` | 루트 컴포넌트(`id='root'`, `basePath='/'`)로 트리를 시작하는 진입점 |

## 상세 명세

### 내부 컴포넌트 `ResolvedChild` (memo)

**Props:**
| 이름 | 타입 |
|------|------|
| `surface` | `SurfaceModel<ReactComponentImplementation>` |
| `id` | `string` |
| `basePath` | `string` |
| `componentModel` | `ComponentModel` |
| `compImpl` | `ReactComponentImplementation` |

`React.memo`로 감싸져 있으며 `displayName`은 `'ResolvedChild'`이다.

**동작:**
1. `compImpl.render`를 `ComponentToRender`로 할당한다.
2. `useMemo`로 `ComponentContext`를 생성한다. 의존성 배열은 `[surface, id, basePath, componentModel]`이며, `componentModel`은 컴포넌트 타입 교체 시 context를 재생성하는 트리거 역할을 한다(실제로 본문에서 사용하지 않지만 eslint 예외 처리).
3. `useCallback`으로 `buildChild` 함수를 생성한다. `childId`와 선택적 `specificPath`를 받아 `<DeferredChild key={childId-path} surface basePath>` JSX를 반환한다. `specificPath`가 없으면 `context.dataContext.path`를 사용한다. 의존성: `[surface, context.dataContext.path]`.
4. `<ComponentToRender context={context} buildChild={buildChild} />`를 렌더링한다.

### 컴포넌트 `DeferredChild` (memo, exported)

**Props:**
| 이름 | 타입 |
|------|------|
| `surface` | `SurfaceModel<ReactComponentImplementation>` |
| `id` | `string` |
| `basePath` | `string` |

`displayName`은 `'DeferredChild'`이다.

**동작:**
1. `useMemo`로 외부 스토어 객체를 생성한다. 스토어는 `subscribe`와 `getSnapshot` 두 메서드를 가진다.
   - 내부 `version` 카운터(`let version = 0`)를 클로저로 유지한다.
   - `subscribe(cb)`: `surface.componentsModel.onCreated`와 `surface.componentsModel.onDeleted`를 구독한다. 각각 `comp.id === id` 또는 `delId === id`일 때 `version++` 후 `cb()`를 호출한다. 구독 해제 함수를 반환한다.
   - `getSnapshot()`: `surface.componentsModel.get(id)` 결과를 가져와 존재하면 `` `${comp.type}-${version}` ``, 없으면 `` `missing-${version}` ``을 반환하여 타입 교체도 재렌더링 트리거로 인식한다.
   - 의존성: `[surface, id]`
2. `useSyncExternalStore(store.subscribe, store.getSnapshot)`을 호출하여 스냅샷 변화 시 리렌더를 구독한다.
3. `surface.componentsModel.get(id)`로 `componentModel`을 조회한다.
4. `componentModel`이 없으면 `<div style={{color:'gray', padding:'4px'}}>[Loading {id}...]</div>`를 렌더링한다.
5. `surface.catalog.components.get(componentModel.type)`으로 `compImpl`을 조회한다.
6. `compImpl`이 없으면 `<div style={{color:'red'}}>Unknown component: {componentModel.type}</div>`를 렌더링한다.
7. 모두 조회 성공 시 `<ResolvedChild surface id basePath componentModel compImpl />`를 렌더링한다.

### 컴포넌트 `A2uiSurface` (exported)

**Props:** `{ surface: SurfaceModel<ReactComponentImplementation> }`

단순히 `<DeferredChild surface={surface} id="root" basePath="/" />`를 렌더링한다. 루트 컴포넌트의 ID는 항상 `'root'`이고 basePath는 `'/'`이다.

## 동작 흐름

1. `<A2uiSurface surface={...} />`가 렌더링되면 id `'root'`인 `DeferredChild`를 시작한다.
2. `DeferredChild`는 `componentsModel`에서 해당 id의 컴포넌트가 나타날 때까지 로딩 상태를 표시하다가, 컴포넌트가 등록되면 자동으로 재렌더링된다.
3. 컴포넌트가 조회되면 카탈로그에서 구현체를 찾아 `ResolvedChild`로 위임한다.
4. `ResolvedChild`는 `ComponentContext`를 생성하고 실제 컴포넌트 구현체(`compImpl.render`)에 context와 `buildChild` 함수를 전달한다.
5. 컴포넌트 구현체가 자식을 렌더링할 때는 `buildChild(childId)`를 호출하며, 이는 새 `DeferredChild`를 생성하여 재귀적으로 트리를 구성한다.
6. 컴포넌트 생성/삭제 이벤트는 `onCreated`/`onDeleted` 구독을 통해 관련 `DeferredChild`만 선택적으로 재렌더링한다.
