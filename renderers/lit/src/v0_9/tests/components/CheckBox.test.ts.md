# renderers/lit/src/v0_9/tests/components/CheckBox.test.ts

## 개요

`a2ui-checkbox` 커스텀 엘리먼트의 유효성 검사 오류 렌더링 동작을 검증하는 단위 테스트 파일이다. `isValid: false`와 `validationErrors` 배열을 가진 컴포넌트 상태에서 Shadow DOM 내 `.error` 요소가 올바른 텍스트와 함께 렌더링되는지 확인한다.

## 의존성

### 외부 패키지
- `node:assert` — `assert`
- `node:test` — `describe`, `it`, `beforeEach`, `after`, `before`
- `@a2ui/web_core/v0_9` — `ComponentContext`, `MessageProcessor`

### 저장소 내부 모듈
- [`../dom-setup.js`](../dom-setup.ts.md) — `setupTestDom`, `teardownTestDom`, `asyncUpdate`
- `../../catalogs/basic/index.js` — `basicCatalog` (동적 import)
- `../../catalogs/basic/components/CheckBox.js` — CheckBox 커스텀 엘리먼트 등록 (동적 import)

## Exports

없음 (테스트 파일)

## 테스트 케이스

### describe: `CheckBox Component`

**픽스처 / 모킹**
- `before` 훅:
  1. `setupTestDom()`으로 JSDOM 전역을 설정한다.
  2. `../../catalogs/basic/index.js`에서 `basicCatalog`를 동적으로 import한다.
  3. `../../catalogs/basic/components/CheckBox.js`를 동적으로 import하여 `a2ui-checkbox` 커스텀 엘리먼트를 등록한다.
- `after` 훅: `teardownTestDom()` 호출.
- `beforeEach` 훅:
  1. `new MessageProcessor([basicCatalog])`로 프로세서를 생성한다.
  2. `createSurface` 메시지(`surfaceId: 'test-surface'`, `catalogId: basicCatalog.id`)를 처리한다.
  3. `updateComponents` 메시지로 컴포넌트 `checkbox_invalid`를 추가한다:
     - `component: 'CheckBox'`, `label: 'Check me'`, `value: false`, `isValid: false`, `validationErrors: ['This is required']`
  4. `processor.model.getSurface('test-surface')`로 `surface`를 획득한다.

---

#### `should render validation error in CheckBox`
- 검증 동작:
  1. `document.createElement('a2ui-checkbox')`로 엘리먼트를 생성하고 `document.body`에 추가한다.
  2. `new ComponentContext(surface, 'checkbox_invalid')`로 컨텍스트를 생성하여 `asyncUpdate`로 엘리먼트에 설정한다.
  3. `el.shadowRoot.querySelector('.error')`로 오류 요소를 찾아 존재 여부를 검증한다.
  4. `errorDiv.textContent.trim()`이 정확히 `'This is required'`인지 검증한다.
  5. `document.body.removeChild(el)`로 정리한다.

## 동작 흐름

JSDOM 전역 설정 → CheckBox 컴포넌트 동적 로딩 → 유효성 오류 상태를 가진 서피스 구성 → DOM 엘리먼트 생성 및 컨텍스트 주입 → Shadow DOM에서 `.error` 요소 검증의 순서로 진행된다.
