# renderers/react/a2ui_explorer/tests/utils/test-utils.tsx

## 개요

브라우저 기반 통합 테스트를 위한 공통 헬퍼 모듈이다. `App` 컴포넌트를 실제 DOM에 마운트/언마운트하는 생명주기 유틸리티와, 렌더링된 DOM에서 요소를 찾는 쿼리 헬퍼를 제공한다. 모든 React 상태 변경은 `act()`로 래핑하여 동기/비동기 사이드 이펙트가 완전히 반영된 뒤 assertion을 수행할 수 있게 한다.

## 의존성

### 외부 패키지
- `react` — `act`
- `react-dom/client` — `createRoot`, `Root`
- `@a2ui/web_core/v0_9` — `A2uiClientAction`

### 저장소 내부 모듈
- [`../../src/App`](../../src/App.tsx.md) — `App` 컴포넌트

## Exports

- `loadExample` (함수) — App 컴포넌트를 DOM에 마운트
- `cleanup` (함수) — 마운트된 컴포넌트와 컨테이너 제거
- `whenSettled` (함수) — React 비동기 상태 정착 대기
- `findButtonByText` (함수) — 텍스트로 버튼 요소 탐색
- `getSurface` (함수) — 렌더링된 첫 번째 서피스 컨테이너 반환

## 상세 명세

### `activeRoot: Root | null` (모듈 레벨 변수)

현재 활성 React 루트 인스턴스를 추적하는 모듈 스코프 변수. 초기값 `null`. `loadExample` 호출 시 갱신되고, `cleanup` 호출 시 `null`로 초기화된다.

### `TEST_CONTAINER_ID: string` (상수)

값: `'test-container'`. 테스트용 컨테이너 `div`의 id 속성 값이다.

### `loadExample(filename: string, onAction?: (action: A2uiClientAction) => void): Promise<HTMLDivElement>`

지정된 예제를 사전 로드한 상태로 `App` 컴포넌트를 DOM에 마운트한다.

**단계**:
1. `cleanup()`을 호출하여 이전 테스트의 잔여 마운트를 정리한다.
2. `document.createElement('div')`로 새 컨테이너를 생성하고 `id`를 `TEST_CONTAINER_ID`로 설정한 뒤 `document.body`에 추가한다.
3. `filename.replace('.json', '')`으로 예제 id를 생성한다.
4. `createRoot(container)`로 React 루트를 생성하고 `activeRoot`에 저장한다.
5. `act()`로 래핑하여 `root.render(<App initialExampleId={id} onAction={onAction} />)`를 호출한다.
6. `whenSettled()`를 `await`하여 React 상태와 메시지 처리가 완전히 반영되도록 기다린다.
7. `container`를 반환한다.

### `cleanup(): Promise<void>`

마운트된 React 컴포넌트와 DOM 컨테이너를 제거한다.

**단계**:
1. `activeRoot`가 있으면 `act(async () => { activeRoot!.unmount(); })`를 `await`한다. `finally` 블록에서 반드시 `activeRoot = null`로 초기화한다.
2. `document.getElementById(TEST_CONTAINER_ID)?.remove()`로 컨테이너 요소를 DOM에서 제거한다.

### `whenSettled(): Promise<void>`

React 상태 업데이트와 비동기 사이드 이펙트가 완전히 반영될 때까지 대기한다.

`act(async () => { await new Promise(resolve => setTimeout(resolve, 0)); })`를 실행한다. `act()`의 비동기 플러시와 macrotask 양보를 조합하여 메시지 프로세서 사이클과 이벤트 콜백이 모두 완료된 뒤 제어권을 반환한다.

### `findButtonByText(root: Element, text: string): HTMLButtonElement`

`root` 내 모든 `button` 요소를 `querySelectorAll`로 수집하고, `textContent?.includes(text)`가 참인 첫 번째 요소를 반환한다. 찾지 못하면 `Button with text "${text}" not found` 메시지로 `Error`를 throw한다.

### `getSurface(root: Element): HTMLElement`

`root.querySelector('[class*="surfaceContainer"]')`로 CSS 모듈 클래스명에 `surfaceContainer`가 포함된 요소를 찾는다. 찾지 못하면 `'surfaceContainer not found in root element'` 메시지로 `Error`를 throw한다. 찾으면 `HTMLElement`로 캐스팅하여 반환한다.

## 동작 흐름

각 테스트의 `beforeEach`에서 `loadExample`을 호출하여 환경을 초기화하고, `afterEach`에서 `cleanup`을 호출하여 격리를 보장한다. `whenSettled`는 비동기 액션 후 assertion 직전에 삽입하여 렌더링 완료를 보장한다. `findButtonByText`와 `getSurface`는 DOM 쿼리의 편의 래퍼로 테스트 코드의 중복을 줄인다.
