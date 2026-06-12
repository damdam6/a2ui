# renderers/react/src/v0_8/core/A2UIProvider.tsx

## 개요

A2UI React 통합의 핵심 Context 제공자 파일이다. `A2UIProvider` 컴포넌트와 세 개의 훅(`useA2UIActions`, `useA2UIState`, `useA2UIContext`)을 정의한다. 성능 최적화를 위해 "안정 액션 컨텍스트"와 "반응형 상태 컨텍스트"를 분리하는 이중 컨텍스트 아키텍처를 사용하며, 메시지 프로세서 초기화와 스타일 주입을 최초 import 시 한 번만 수행한다.

## 의존성

### 외부 패키지
- `react` — `createContext`, `useContext`, `useRef`, `useState`, `useMemo`, `ReactNode`
- `@a2ui/web_core/types/types` — `Types` 네임스페이스 전체 (타입 전용)
- `@a2ui/web_core/data/model-processor` — `A2uiMessageProcessor` 클래스

### 저장소 내부 모듈
- [`./store`](./store.ts.md) — `A2UIContextValue`, `A2UIActions` 인터페이스
- `../theme/ThemeContext` — `ThemeProvider` 컴포넌트
- `../registry/defaultCatalog` — `initializeDefaultCatalog` 함수
- `../styles` — `injectStyles` 함수
- [`../types`](../types.ts.md) — `OnActionCallback` 타입

## Exports

| 이름 | 종류 |
|------|------|
| `A2UIProviderProps` | 인터페이스 |
| `A2UIProvider` | React 함수 컴포넌트 |
| `useA2UIActions` | 훅 함수 |
| `useA2UIState` | 훅 함수 |
| `useA2UIContext` | 훅 함수 |

## 상세 명세

### 모듈 레벨 변수

#### `initialized: boolean`
초깃값 `false`. `ensureInitialized()`가 최초 호출 시 `true`로 바꾸어 중복 초기화를 방지한다.

#### `A2UIActionsContext`
`createContext<A2UIActions | null>(null)`로 생성된 Context. 안정적인 액션 객체를 보유하며, 이 컨텍스트를 구독하는 컴포넌트는 액션 참조가 변하지 않으므로 불필요한 리렌더링이 발생하지 않는다.

#### `A2UIStateContext`
`createContext<{version: number} | null>(null)`로 생성된 Context. `version`이 변경될 때마다 이 컨텍스트를 구독하는 컴포넌트를 리렌더링한다.

### `ensureInitialized(): void`

`initialized` 플래그가 `false`일 때만 `initializeDefaultCatalog()`와 `injectStyles()`를 호출하고 플래그를 `true`로 설정한다. 이후 호출에서는 아무것도 하지 않는다.

### `A2UIProviderProps` 인터페이스

| 필드 | 타입 | 필수 여부 | 설명 |
|------|------|---------|------|
| `onAction` | `OnActionCallback` | 선택 | 버튼 클릭 등 사용자 액션 발생 시 호출되는 콜백 |
| `theme` | `Types.Theme` | 선택 | 테마 설정; 미제공 시 기본 테마 사용 |
| `children` | `ReactNode` | 필수 | 자식 컴포넌트 |

### `A2UIProvider` 컴포넌트

**시그니처:** `function A2UIProvider({onAction, theme, children}: A2UIProviderProps): JSX.Element`

**동작 단계:**
1. `ensureInitialized()` 호출로 카탈로그와 스타일을 한 번만 초기화한다.
2. `useRef`로 `processorRef`를 생성하고 최초 렌더 시 `new A2uiMessageProcessor()`를 할당한다. 이후 렌더링에서는 기존 인스턴스를 재사용한다.
3. `useState(0)`으로 `version` 상태와 `setVersion` 세터를 만든다. 이 카운터가 변경될 때만 상태 컨텍스트 구독자가 리렌더링된다.
4. `onActionRef`를 `useRef`로 생성하고 매 렌더마다 최신 `onAction` 콜백으로 업데이트한다. 이렇게 하면 오래된 클로저 문제 없이 항상 최신 콜백이 호출된다.
5. `actionsRef`를 `useRef`로 생성하고 최초 렌더 시에만 `A2UIActions` 구현 객체를 할당한다. 객체 내부의 각 메서드는 다음과 같이 동작한다:
   - `processMessages`: `processor.processMessages(messages)` 호출 후 `setVersion(v => v + 1)`로 리렌더링 트리거.
   - `setData`: `processor.setData(...)` 호출 후 `setVersion(v => v + 1)`.
   - `dispatch`: `onActionRef.current`가 존재하면 `void onActionRef.current(message)` 호출.
   - `clearSurfaces`: `processor.clearSurfaces()` 후 `setVersion(v => v + 1)`.
   - `getSurface`: `processor.getSurfaces().get(surfaceId)` 반환.
   - `getSurfaces`: `processor.getSurfaces()` 반환.
   - `getData`: `processor.getData(node, path, surfaceId)` 반환.
   - `resolvePath`: `processor.resolvePath(path, dataContextPath)` 반환.
6. `stateValue`를 `useMemo(() => ({version}), [version])`으로 메모이제이션한다.
7. `A2UIActionsContext.Provider` > `A2UIStateContext.Provider` > `ThemeProvider` 순으로 중첩하여 `children`을 감싸 반환한다.

### `useA2UIActions(): A2UIActions`

`useContext(A2UIActionsContext)`를 호출한다. 결과가 `null`이면 `'useA2UIActions must be used within an A2UIProvider'` 메시지로 에러를 throw한다. 이 훅은 상태 컨텍스트를 구독하지 않으므로 `version` 변경 시 리렌더링을 유발하지 않는다.

### `useA2UIState(): {version: number}`

`useContext(A2UIStateContext)`를 호출한다. 결과가 `null`이면 `'useA2UIState must be used within an A2UIProvider'` 메시지로 에러를 throw한다. 이 훅을 사용하는 컴포넌트는 `version`이 바뀔 때마다 리렌더링된다.

### `useA2UIContext(): A2UIContextValue`

`useA2UIActions()`와 `useA2UIState()`를 모두 호출한 뒤, `useMemo`로 두 값을 합성한 `A2UIContextValue` 객체를 반환한다. `processor` 필드는 `null as unknown as Types.MessageProcessor`로 설정(직접 노출하지 않음)하고, `onAction`은 `null`로 설정한다. 메모이제이션 의존성은 `[actions, state.version]`이다.

## 동작 흐름

```
최초 import
  → ensureInitialized() → initializeDefaultCatalog() + injectStyles()

A2UIProvider 렌더
  → processorRef 초기화 (한 번만)
  → actionsRef 초기화 (한 번만)
  → onActionRef 최신값으로 갱신 (매 렌더)
  → stateValue = {version} 메모이제이션

서버 메시지 수신
  → actions.processMessages(messages)
    → processor.processMessages(messages)
    → setVersion(v+1) → 상태 컨텍스트 구독자 리렌더링

사용자 액션 발생
  → actions.dispatch(message)
    → onActionRef.current(message) 호출 (최신 콜백 사용)
```
