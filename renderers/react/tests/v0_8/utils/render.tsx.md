# renderers/react/tests/v0_8/utils/render.tsx

## 개요

v0.8 테스트에서 React 컴포넌트를 렌더링하기 위한 공통 래퍼 컴포넌트 모듈이다. `TestRenderer`는 메시지 배열을 받아 A2UI 상태를 초기화하고 지정한 surface를 렌더링하며, `TestWrapper`는 `A2UIProvider`로 감싸 테스트 환경의 컨텍스트를 설정한다. 두 컴포넌트를 항상 중첩하여 사용한다.

## 의존성

### 외부 패키지
- `react`: `React`, `useEffect`, `ReactNode` 타입
- `@a2ui/web_core/types/types`: `Types` 네임스페이스 (type-only import)

### 저장소 내부 모듈
- `../../../src/v0_8` — `A2UIProvider`, `A2UIRenderer`, `useA2UI` (v0.8 공개 API)

## Exports

| 이름 | 종류 |
|---|---|
| `TestRenderer` | React 함수 컴포넌트 |
| `TestWrapper` | React 함수 컴포넌트 |

## 상세 명세

### `TestRenderer({ messages, surfaceId? })`

- **props**
  - `messages: Types.ServerToClientMessage[]` — 처리할 메시지 배열
  - `surfaceId?: string` — 렌더링할 surface ID, 기본값 `'@default'`
- **반환**: `<A2UIRenderer surfaceId={surfaceId} />` JSX
- **동작 로직**
  1. `useA2UI()` 훅을 호출하여 `processMessages` 함수를 꺼낸다.
  2. `useEffect`에서 `processMessages(messages)`를 실행한다. 의존성 배열은 `[messages, processMessages]`로, messages나 processMessages가 변경되면 재실행된다.
  3. `<A2UIRenderer surfaceId={surfaceId} />`를 렌더링하여 해당 surface의 UI를 화면에 그린다.

### `TestWrapper({ children, onAction?, theme? })`

- **props**
  - `children: ReactNode` — 내부에 렌더링할 자식 컴포넌트
  - `onAction?: (action: Types.A2UIClientEventMessage) => void` — 사용자 인터랙션 이벤트 핸들러 (선택)
  - `theme?: Types.Theme` — 커스텀 테마 객체 (선택)
- **반환**: `<A2UIProvider onAction={onAction} theme={theme}>{children}</A2UIProvider>` JSX
- **동작 로직**: `A2UIProvider`에 `onAction`과 `theme`을 전달하여 테스트 환경의 컨텍스트를 구성한다. 자체 로직은 없으며 단순 위임 패턴이다.

## 동작 흐름

전형적인 사용 패턴:
1. 테스트 파일에서 `createSimpleMessages(...)` 등으로 메시지 배열 생성.
2. `<TestWrapper [theme=...] [onAction=...]>` 안에 `<TestRenderer messages={...} [surfaceId=...] />`를 중첩하여 `render()` 호출.
3. `TestWrapper`가 `A2UIProvider` 컨텍스트를 제공하고, `TestRenderer`가 `useEffect`에서 메시지를 처리한 뒤 `A2UIRenderer`가 DOM을 생성한다.
