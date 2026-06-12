# renderers/react/a2ui_explorer/src/App.tsx

## 개요

A2UI React 탐색기의 루트 UI 컴포넌트이다. 예제 목록을 사이드바에 표시하고, 선택된 예제의 A2UI 메시지를 단계별로 처리하여 `A2uiSurface`로 렌더링한다. 오른쪽 패널에서 라이브 데이터 모델과 액션 로그를 실시간으로 확인할 수 있다.

## 의존성

### 외부 패키지
- `react` — `useState`, `useEffect`, `useSyncExternalStore`, `useCallback`, `useRef`
- `@a2ui/web_core/v0_9` — `MessageProcessor`, `SurfaceModel`, `A2uiClientAction`
- `@a2ui/react/v0_9` — `basicCatalog`, `A2uiSurface`, `MarkdownContext`, `ReactComponentImplementation`
- `@a2ui/markdown-it` — `renderMarkdown`

### 저장소 내부 모듈
- [`./examples`](./examples.ts.md) — `getDemoItems` 함수
- `./App.module.css` — CSS 모듈 스타일

## Exports

- `AppProps` (interface) — 컴포넌트 props 타입
- `App` (함수 컴포넌트) — 메인 탐색기 컴포넌트

## 상세 명세

### `AppProps` (interface)

| 필드 | 타입 | 설명 |
|---|---|---|
| `initialExampleId?` | `string` | 초기 마운트 시 선택할 예제 id. 테스트 전용 |
| `onAction?` | `(action: A2uiClientAction) => void` | 디스패치된 액션을 가로채는 콜백. 테스트 전용 |

### `LogEntry` (interface, 비공개)

| 필드 | 타입 | 설명 |
|---|---|---|
| `time` | `string` | 액션이 인터셉트된 ISO 타임스탬프 |
| `action` | `A2uiClientAction` | 인터셉트된 클라이언트 액션 객체 |

### `demoItems` (모듈 레벨 상수)

`getDemoItems()`를 모듈 로드 시점에 한 번 호출하여 캐시한 `DemoItem[]` 배열이다.

### `DataModelViewer` (내부 컴포넌트)

**시그니처**: `({surface}: {surface: SurfaceModel<ReactComponentImplementation>}) => JSX.Element`

지정된 `surface`의 데이터 모델을 실시간으로 표시하는 내부 컴포넌트이다.

- `useSyncExternalStore`를 통해 `surface.dataModel.subscribe('/', callback)`으로 구독하고, `surface.dataModel.get('/')`의 JSON 직렬화 결과를 스냅샷으로 반환한다.
- `subscribeHook`과 `getSnapshot` 모두 `useCallback`으로 메모이제이션되며 `surface`를 의존성으로 갖는다.
- `surface.id`와 JSON 데이터를 `<pre>` 태그로 렌더링한다.

### `App` (함수 컴포넌트)

**시그니처**: `({initialExampleId, onAction}: AppProps) => JSX.Element`

**상태(state)**:
- `selectedExampleId: string` — 현재 선택된 예제 id. 초기값은 `initialExampleId ?? demoItems[0].id`.
- `logs: LogEntry[]` — 캡처된 액션 로그 목록
- `processor: MessageProcessor<ReactComponentImplementation> | null` — 현재 활성 메시지 프로세서
- `surfaces: string[]` — 활성 서피스 id 목록
- `currentMessageIndex: number` — 현재까지 처리된 마지막 메시지의 인덱스. 초기값 `-1`.

**`onActionRef`**: `onAction` prop의 최신 참조를 `useRef`로 유지하여 `resetProcessor`의 클로저에서 항상 최신 콜백을 사용한다.

**`resetProcessor(advanceToEnd: boolean = false): void`**

`setProcessor` 함수형 업데이트 내에서 실행된다.

1. 이전 `processor`가 존재하면 `prevProcessor.model.dispose()`를 호출한다.
2. 새 `MessageProcessor`를 `[basicCatalog]`와 액션 핸들러로 생성한다. 핸들러는 `setLogs`로 로그를 추가하고 `onActionRef.current`를 호출한다.
3. `advanceToEnd`가 `true`이고 `selectedItem?.messages`가 존재하면 `newProcessor.processMessages(msgs)`를 즉시 호출한다.
4. `setLogs([])`, `setSurfaces([])`로 파생 상태를 초기화한다.
5. `advanceToEnd`가 `true`면 `currentMessageIndex`를 `msgs.length - 1`로, 아니면 `-1`로 설정한다.

**`useEffect` — 예제 선택 변경 처리**

`[selectedExampleId, resetProcessor]`를 의존성으로 가진다. 마운트 및 예제 변경 시 `resetProcessor(true)`를 호출한다. 클린업 함수에서 `setProcessor`를 통해 현재 프로세서의 `model.dispose()`를 호출하고 `null`로 설정한다.

**`useEffect` — 서피스 구독**

`[processor]`를 의존성으로 가진다. `processor`가 없으면 `setSurfaces([])`하고 반환한다. 있으면 `processor.model.surfacesMap`에서 서피스 id를 추출하는 `updateSurfaces` 함수를 정의하고 즉시 호출한다. `processor.model.onSurfaceCreated`와 `onSurfaceDeleted`에 `updateSurfaces`를 구독하고, 클린업 함수에서 두 구독을 해제한다.

**`advanceToMessage(index: number): void`**

`processor`나 `msgs`가 없으면 즉시 반환한다. `msgs.slice(currentMessageIndex + 1, index + 1)`로 처리할 메시지를 추출하고, 길이가 0보다 크면 `processor.processMessages(messagesToProcess)`를 호출한 뒤 `setCurrentMessageIndex(index)`로 인덱스를 갱신한다.

**`handleReset(): void`**

`resetProcessor(false)`를 호출하여 프로세서를 초기 상태로 되돌린다.

**렌더 구조**:
- `header`: 제목, 부제목, 현재 메시지 인덱스 표시, Reset 버튼
- `main.navPane`: `demoItems` 전체를 버튼 목록으로 렌더링. 선택된 항목에 `styles.active` 클래스 적용
- `main.galleryPane`:
  - 서피스 렌더링 영역: 각 서피스를 `MarkdownContext.Provider`(value=`renderMarkdown`)로 감싸고 `A2uiSurface`로 렌더링
  - 메시지 스테퍼: 메시지 목록을 표시하고 처리되지 않은 메시지에 "Advance" 버튼 제공
- `main.inspectorPane`:
  - Data Model 섹션: 각 서피스에 `DataModelViewer` 렌더링
  - Action Logs 섹션: `logs` 배열을 타임스탬프와 함께 렌더링

## 동작 흐름

1. 모듈 로드 시 `getDemoItems()`로 `demoItems`를 구성한다.
2. `App` 마운트 시 첫 번째 예제(또는 `initialExampleId`)를 선택하고 `resetProcessor(true)`로 전체 메시지를 즉시 처리한다.
3. 사용자가 예제를 전환하면 `selectedExampleId` 상태가 변경되어 `resetProcessor(true)`가 다시 실행된다.
4. "Advance" 버튼 클릭 시 해당 인덱스까지의 메시지가 순차 처리되며, 서피스 상태와 데이터 모델 뷰어가 실시간으로 갱신된다.
5. 액션 발생 시 `LogEntry`가 추가되고 `onAction` prop이 있으면 함께 호출된다.
