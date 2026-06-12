# renderers/lit/src/v0_9/tests/a2ui-lit-element.test.ts

## 개요

`A2uiLitElement` 베이스 클래스의 동작을 검증하는 테스트 파일이다. JSDOM 환경에서 컨트롤러 생성/소멸 생명주기, 컨텍스트 변경 시 재생성 동작, 서피스·컴포넌트 제거 시의 안전한 렌더링 종료(nothing 반환)를 `node:test` 프레임워크로 테스트한다.

## 의존성

### 외부 패키지
- `node:assert`: `assert`
- `node:test`: `describe`, `it`, `beforeEach`, `after`, `before`
- `@a2ui/web_core/v0_9`: `ComponentContext`, `MessageProcessor`

### 저장소 내부 모듈
- [`./dom-setup.js`](./dom-setup.ts.md) — `setupTestDom`, `teardownTestDom`, `asyncUpdate`
- [`../a2ui-lit-element.js`](../a2ui-lit-element.ts.md) — `A2uiLitElement` (동적 임포트)
- [`../catalogs/basic/index.js`](../catalogs/basic/index.ts.md) — `basicCatalog` (동적 임포트)

## 테스트 케이스

### 픽스처 및 설정

**모듈 레벨 변수**
- `controllerCreatedCount: number` — `createController()` 호출 횟수 추적
- `disposedCount: number` — 컨트롤러의 `dispose()` 호출 횟수 추적
- `lastRenderResult: any` — `TestA2uiElement.render()`에서 `renderNode('child_id')`의 반환값을 저장, 렌더링 동작 검증에 사용

**`before()` 훅**
- `setupTestDom()`으로 JSDOM 환경을 초기화한다.
- `A2uiLitElement`와 `basicCatalog`를 동적 임포트한다(JSDOM 글로벌 설정 이후 LitElement 평가 방지).
- `A2uiLitElement<any>`를 상속하는 `TestA2uiElement` 클래스를 정의한다:
  - `createController()`: `controllerCreatedCount`를 증가시키고 `{ dispose: () => { disposedCount++ } }` 형태의 mock 컨트롤러를 반환한다.
  - `render()`: `this.renderNode('child_id')`를 호출하고 결과를 `lastRenderResult`에 저장한 뒤 반환한다.
- `customElements.define('test-a2ui-element', TestA2uiElement)`로 등록한다.

**`after()` 훅**
- `teardownTestDom()`으로 JSDOM 환경을 정리한다.

**`beforeEach()` 훅**
- `controllerCreatedCount`와 `disposedCount`를 0으로 초기화한다.
- `new MessageProcessor([basicCatalog])`로 프로세서를 생성한다.
- `createSurface`, `updateComponents`(루트: `Text/'Root'`, 자식: `child_id`/`Text/'Child'`), 두 개의 `updateDataModel` 메시지(`/: {myData: 'hello'}`, `/child_id: {myData: 'world'}`)를 처리한다.
- `processor.model.getSurface('test-surface')`로 `surface`를 가져온다.

---

### `'should create controller when context is set'`
- **목적**: `context` 프로퍼티가 설정될 때 `createController()`가 한 번 호출되는지 검증한다.
- **방법**: `test-a2ui-element`를 생성하고 body에 추가한다. 초기 상태(`context = undefined`)에서 카운트가 0임을 확인한 뒤, `asyncUpdate`로 `context = new ComponentContext(surface, 'root')`를 설정한다.
- **검증**: `controllerCreatedCount === 1` (`assert.strictEqual`).

---

### `'should dispose old controller and create new one when context changes'`
- **목적**: `context` 프로퍼티가 다른 값으로 변경될 때 기존 컨트롤러가 소멸(`dispose`)되고 새 컨트롤러가 생성되는지 검증한다.
- **방법**:
  1. `asyncUpdate`로 `context1 = new ComponentContext(surface, 'root')`를 설정한다.
  2. `controllerCreatedCount === 1`, `disposedCount === 0`임을 확인한다.
  3. `asyncUpdate`로 `context2 = new ComponentContext(surface, 'child_id')`로 교체한다.
- **검증**: `disposedCount === 1`, `controllerCreatedCount === 2` (`assert.strictEqual`).

---

### `'should return nothing when component is removed from surface'`
- **목적**: `surface.componentsModel`에서 컴포넌트가 제거된 후 렌더링 시 `nothing`을 반환하고 오류를 던지지 않는지 검증한다.
- **방법**:
  1. `context = new ComponentContext(surface, 'root')`를 설정한다.
  2. `surface.componentsModel.removeComponent('root')`로 컴포넌트를 제거한다.
  3. `asyncUpdate(el, e => { e.requestUpdate() })`로 강제 재렌더링을 트리거한다.
- **검증**: `lastRenderResult === nothing` (`assert.strictEqual`). `nothing`을 동적 임포트(`await import('lit')`)로 가져와 비교한다.

---

### `'should return nothing when surface is disposed'`
- **목적**: 서피스가 dispose된 후 렌더링 시 `nothing`을 반환하고 오류를 던지지 않는지 검증한다.
- **방법**:
  1. `context = new ComponentContext(surface, 'root')`를 설정한다.
  2. `surface.dispose()`로 서피스를 소멸시킨다.
  3. `asyncUpdate(el, e => { e.requestUpdate() })`로 강제 재렌더링을 트리거한다.
- **검증**: `lastRenderResult === nothing` (`assert.strictEqual`). `nothing`을 동적 임포트로 가져와 비교한다.
