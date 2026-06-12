# renderers/lit/src/v0_9/tests/a2ui-controller.test.ts

## 개요

`A2uiController`의 동작을 검증하는 테스트 파일이다. JSDOM 환경에서 Lit 엘리먼트와 컨트롤러의 초기화, 반응형 업데이트, 구독 해제 동작을 `node:test` 프레임워크로 테스트한다.

## 의존성

### 외부 패키지
- `node:assert`: `assert`
- `node:test`: `describe`, `it`, `beforeEach`, `mock`, `after`, `before`
- `@a2ui/web_core/v0_9`: `MessageProcessor`, `ComponentContext`
- `@a2ui/web_core/v0_9/basic_catalog`: `TextApi`

### 저장소 내부 모듈
- [`./dom-setup.js`](./dom-setup.ts.md) — `setupTestDom`, `teardownTestDom`, `asyncUpdate`
- [`../a2ui-controller.js`](../a2ui-controller.ts.md) — `A2uiController`
- [`../catalogs/basic/index.js`](../catalogs/basic/index.ts.md) — `basicCatalog` (동적 임포트)
- [`../a2ui-lit-element.js`](../a2ui-lit-element.ts.md) — `A2uiLitElement` (동적 임포트)

## 테스트 케이스

### 픽스처 및 설정

**`before()` 훅**
- `setupTestDom()`으로 JSDOM 환경을 초기화한다.
- `A2uiLitElement`와 `basicCatalog`를 동적 임포트한다(JSDOM 글로벌 설정 이후 LitElement 평가 방지).
- `A2uiLitElement<any>`를 상속하는 `TestMockHost` 클래스를 정의하고 `createController()`를 오버라이드하여 `TextApi`로 `A2uiController`를 생성하고 `this.testController`에 저장한다. `customElements.define('test-mock-host', TestMockHost)`로 등록한다.

**`after()` 훅**
- `teardownTestDom()`으로 JSDOM 환경을 정리한다.

**`beforeEach()` 훅**
- `new MessageProcessor([basicCatalog])`로 프로세서를 생성한다.
- `createSurface` 메시지(`surfaceId: 'test-surface'`, `catalogId: basicCatalog.id`)와 `updateComponents` 메시지로 `id: 'test-comp'`, `component: 'Text'`, `text: 'Initial'`인 초기 컴포넌트를 생성한다.
- `processor.model.getSurface('test-surface')`로 `surface`를 가져오고 `new ComponentContext(surface, 'test-comp')`로 `context`를 생성한다.

**`createMockHost(context: ComponentContext)` 헬퍼 함수**
- `document.createElement('test-mock-host')`로 엘리먼트를 생성하고 `document.body`에 추가한다.
- `asyncUpdate(mockHost, host => { host.context = context; })`로 컨텍스트를 설정하고 Lit의 반응형 업데이트 사이클이 완료될 때까지 대기한다.
- 생성된 `mockHost`를 반환한다.

---

### `'should initialize with correct props and bind to context'`
- **목적**: 컨트롤러가 초기화 시 올바른 props를 갖는지 및 `addController`가 호출되는지 검증한다.
- **방법**: `createMockHost` 헬퍼 대신 직접 엘리먼트를 생성한 뒤 `mock.method(mockHost, 'addController')`로 스파이를 설치하고 컨텍스트를 설정한다.
- **검증**:
  - `mockHost.addController`가 1번 이상 호출되었는지 확인(`assert.ok(mock.calls.length >= 1)`).
  - `controller.props.text === 'Initial'`인지 확인(`assert.strictEqual`).

---

### `'should request update on host when data changes'`
- **목적**: 컴포넌트 모델의 데이터가 변경될 때 컨트롤러가 호스트 엘리먼트의 `requestUpdate()`를 호출하는지 검증한다.
- **방법**:
  1. `createMockHost(context)`로 첫 번째 호스트를 생성하고 `mock.method(mockHost, 'requestUpdate')`를 설치한다.
  2. `asyncUpdate`로 `updateComponents`(새 컴포넌트 `test-comp-2`, `text: {path: '/myText'}`)와 `updateDataModel`(`/myText: 'Updated'`) 메시지를 처리한다.
  3. 기존 컨트롤러를 `controller.dispose()`로 정리하고 `new ComponentContext(surface, 'test-comp-2')`로 두 번째 컨텍스트를 만든다.
  4. 두 번째 `createMockHost(context2)`로 새 호스트를 생성하고 `mock.method`를 설치한다.
  5. 다시 `asyncUpdate`로 `/myText: 'Update2'`를 업데이트한다.
- **검증**:
  - `controller2.props.text === 'Updated'` (초기 props 확인).
  - `controller2.props.text === 'Update2'` (업데이트 반영 확인).
  - `mockHost2.requestUpdate` mock의 호출 횟수가 0보다 크다는 것 확인.
  - 마지막에 `controller2.dispose()` 호출.

---

### `'should unsubscribe when host disconnected'`
- **목적**: 호스트가 연결 해제되면 컨트롤러가 구독을 해제하고 이후 데이터 변경 시 `requestUpdate()`를 호출하지 않는지 검증한다.
- **방법**:
  1. `createMockHost(context)`로 호스트를 생성하고 `mock.method(mockHost, 'requestUpdate')`를 설치한다.
  2. `controller.hostDisconnected()`를 직접 호출하여 연결 해제 상태를 시뮬레이션한다.
  3. `initialCalls`로 현재 호출 횟수를 기록한다.
  4. `asyncUpdate`로 `updateComponents`(`test-comp` 텍스트를 `'Another'`로 변경)를 처리한다.
- **검증**:
  - `controller.props.text !== 'Another'` — props가 업데이트되지 않는다(`assert.notStrictEqual`).
  - `requestUpdate` 호출 횟수가 `initialCalls`와 동일하다 — 추가 호출 없음(`assert.strictEqual`).
