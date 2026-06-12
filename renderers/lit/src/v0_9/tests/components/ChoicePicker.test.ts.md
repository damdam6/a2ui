# renderers/lit/src/v0_9/tests/components/ChoicePicker.test.ts

## 개요

`a2ui-choicepicker` 커스텀 엘리먼트의 두 가지 핵심 동작을 검증하는 단위 테스트 파일이다. `displayStyle: 'chips'` 설정 시 칩 버튼이 렌더링되는지, 그리고 `filterable: true` 설정 시 `filter` 속성 변경에 따라 옵션 목록이 동적으로 필터링되는지 확인한다.

## 의존성

### 외부 패키지
- `node:assert` — `assert`
- `node:test` — `describe`, `it`, `beforeEach`, `after`, `before`
- `@a2ui/web_core/v0_9` — `ComponentContext`, `MessageProcessor`

### 저장소 내부 모듈
- [`../dom-setup.js`](../dom-setup.ts.md) — `setupTestDom`, `teardownTestDom`, `asyncUpdate`
- `../../catalogs/basic/index.js` — `basicCatalog` (동적 import)
- `../../catalogs/basic/components/ChoicePicker.js` — ChoicePicker 커스텀 엘리먼트 등록 (동적 import)

## Exports

없음 (테스트 파일)

## 테스트 케이스

### describe: `ChoicePicker Component`

**픽스처 / 모킹**
- `before` 훅:
  1. `setupTestDom()`으로 JSDOM 전역을 설정한다.
  2. `../../catalogs/basic/index.js`에서 `basicCatalog`를 동적으로 import한다.
  3. `../../catalogs/basic/components/ChoicePicker.js`를 동적으로 import하여 `a2ui-choicepicker` 커스텀 엘리먼트를 등록한다.
- `after` 훅: `teardownTestDom()` 호출.
- `beforeEach` 훅:
  1. `new MessageProcessor([basicCatalog])`로 프로세서를 생성한다.
  2. `createSurface` 메시지(`surfaceId: 'test-surface'`, `catalogId: basicCatalog.id`)를 처리한다.
  3. `updateComponents` 메시지로 두 개의 컴포넌트를 추가한다:
     - `choice_picker_chips`: `component: 'ChoicePicker'`, `label: 'Pick chips'`, `options: [{label: 'Apple', value: 'apple'}, {label: 'Banana', value: 'banana'}]`, `value: []`, `displayStyle: 'chips'`
     - `choice_picker_filterable`: `component: 'ChoicePicker'`, `label: 'Filter me'`, `options: [{label: 'Apple', value: 'apple'}, {label: 'Banana', value: 'banana'}]`, `value: []`, `filterable: true`
  4. `processor.model.getSurface('test-surface')`로 `surface`를 획득한다.

---

#### `should render chips when displayStyle is chips`
- 검증 동작:
  1. `document.createElement('a2ui-choicepicker')`로 엘리먼트를 생성하고 DOM에 추가한다.
  2. `new ComponentContext(surface, 'choice_picker_chips')`로 컨텍스트를 생성하여 `asyncUpdate`로 설정한다.
  3. `el.shadowRoot.querySelectorAll('button.chip')`로 칩 버튼들을 조회하여 개수가 `2`인지 검증한다.
  4. 첫 번째 버튼의 `textContent.trim()`이 `'Apple'`인지 검증한다.
  5. `document.body.removeChild(el)`로 정리한다.

#### `should filter options when filterable is true`
- 검증 동작:
  1. `document.createElement('a2ui-choicepicker')`로 엘리먼트를 생성하고 DOM에 추가한다.
  2. `new ComponentContext(surface, 'choice_picker_filterable')`로 컨텍스트를 설정한다.
  3. 초기 상태에서 `el.shadowRoot.querySelectorAll('label').length`가 `3`(옵션 2개 + 메인 레이블 1개)임을 검증한다.
  4. `asyncUpdate`로 `e.filter = 'app'`을 설정하여 필터를 적용한다.
  5. 필터 후 `label` 개수가 `2`(Apple 1개 + 메인 레이블 1개)로 줄어들었음을 검증한다.
  6. `document.body.removeChild(el)`로 정리한다.

## 동작 흐름

JSDOM 전역 설정 → ChoicePicker 컴포넌트 동적 로딩 → 두 가지 표시 방식을 가진 서피스 구성 → 각 테스트에서 독립 DOM 엘리먼트 생성 및 컨텍스트 주입 → Shadow DOM 쿼리로 렌더링 결과 검증의 순서로 진행된다.
