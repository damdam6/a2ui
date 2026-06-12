# renderers/lit/src/v0_9/tests/basic-catalog-a2ui-lit-element.test.ts

## 개요

`BasicCatalogA2uiLitElement` 기반 엘리먼트가 서피스 테마에서 CSS 커스텀 프로퍼티를 올바르게 적용하고 제거하는지 검증하는 통합 테스트 파일이다. 테마의 `primaryColor`가 `--a2ui-color-primary` 및 파생 변수들에 반영되는지, 그리고 테마가 없는 컨텍스트로 전환할 때 해당 변수들이 제거되는지를 확인한다. JSDOM 환경에서 실행하기 위해 [`./dom-setup.ts`](./dom-setup.ts.md)를 사용한다.

## 의존성

### 외부 패키지
- `node:assert` — `assert`
- `node:test` — `describe`, `it`, `beforeEach`, `after`, `before`
- `@a2ui/web_core/v0_9` — `ComponentContext`, `MessageProcessor`, `Catalog`, `ComponentApi`, `SurfaceModel`

### 저장소 내부 모듈
- [`./dom-setup.js`](./dom-setup.ts.md) — `setupTestDom`, `teardownTestDom`, `asyncUpdate`
- `../types.js` — [`LitComponentApi`](../types.ts.md)
- `../a2ui-controller.js` — `A2uiController`
- `../catalogs/basic/basic-catalog-a2ui-lit-element.js` — `BasicCatalogA2uiLitElement` (동적 import)
- `../catalogs/basic/index.js` — `basicCatalog` (동적 import)

## Exports

없음 (테스트 파일)

## 테스트 케이스

### describe: `BasicCatalogA2uiLitElement`

**픽스처 / 모킹**
- `before` 훅:
  1. `setupTestDom()`으로 JSDOM 전역을 주입한다.
  2. `../catalogs/basic/basic-catalog-a2ui-lit-element.js`를 동적으로 import하여 `BasicCatalogA2uiLitElement` 클래스를 획득한다.
  3. `../catalogs/basic/index.js`에서 `basicCatalog`를 획득한다.
  4. `BasicCatalogA2uiLitElement<ComponentApi>`를 상속하는 `TestBasicElement` 클래스를 익명으로 정의한다.
     - `createController()`: `props: {}`, `dispose: () => {}`를 가진 `A2uiController<ComponentApi>` 모의 객체를 반환한다.
     - `render()`: `null`을 반환한다.
  5. `customElements.define('test-basic-element', TestBasicElement)`로 등록한다.
- `after` 훅: `teardownTestDom()` 호출.
- `beforeEach` 훅:
  1. `new MessageProcessor<LitComponentApi>([basicCatalog])`로 프로세서를 생성한다.
  2. `createSurface` 메시지(서피스 ID: `'test-surface'`, 카탈로그 ID: `basicCatalog.id`, 테마: `{ primaryColor: '#ff0000' }`)를 처리한다.
  3. `updateComponents` 메시지(컴포넌트: `id: 'root'`, `component: 'Text'`, `text: 'Root'`)를 처리한다.
  4. `processor.model.getSurface('test-surface')`로 `surface`를 획득한다.

---

#### `should apply primary color from theme`
- 검증 동작: `test-basic-element` 커스텀 엘리먼트를 생성하고, `new ComponentContext(surface, 'root')`로 컨텍스트를 설정한 뒤 `asyncUpdate`로 업데이트를 완료한다. 이후 엘리먼트 `style`에 다음 CSS 변수들이 정확한 값으로 설정되었는지 검증한다:
  - `--a2ui-color-primary`: `'#ff0000'`
  - `--a2ui-color-primary-light`: `'color-mix(in oklab, var(--a2ui-color-primary) 85%, white)'`
  - `--a2ui-color-primary-dark`: `'color-mix(in oklab, var(--a2ui-color-primary) 85%, black)'`
  - `--a2ui-color-primary-hover`: `'light-dark(var(--a2ui-color-primary-dark), var(--a2ui-color-primary-light))'`

#### `should remove primary color when theme changes or is missing`
- 검증 동작:
  1. 먼저 테마가 있는 컨텍스트(`primaryColor: '#ff0000'`)를 설정하여 CSS 변수가 적용되었음을 중간 검증한다.
  2. `{ ...surface, theme: {} }`로 테마가 없는 서피스 모의 객체를 생성하고, 이를 사용한 새 `ComponentContext`로 엘리먼트 컨텍스트를 교체한다.
  3. `asyncUpdate` 후 `--a2ui-color-primary`, `--a2ui-color-primary-light`, `--a2ui-color-primary-dark`, `--a2ui-color-primary-hover` 모두 `''`(빈 문자열)로 제거되었는지 검증한다.
- 비고: 서피스 테마는 생성 후 불변이지만, 엘리먼트가 재사용될 때의 강건한 정리 동작을 보장하기 위한 테스트이다.

## 동작 흐름

JSDOM 전역 설정 → Lit 컴포넌트 동적 로딩 → 테스트용 커스텀 엘리먼트 등록 순으로 초기화한다. 각 테스트에서 실제 DOM 엘리먼트를 생성하고 `ComponentContext`를 통해 서피스 테마를 주입하여, CSS 커스텀 프로퍼티 적용 및 제거 동작을 인라인 스타일로 검증한다.
