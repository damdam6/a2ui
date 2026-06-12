# renderers/lit/src/v0_9/tests/a2ui-surface.test.ts

## 개요

`A2uiSurface` 커스텀 엘리먼트의 렌더링 생명주기를 검증하는 통합 테스트 파일이다. 서피스 모델이 없을 때 빈 출력, 루트 컴포넌트가 없는 서피스에 대한 로딩 상태, 루트 컴포넌트가 추가된 이후 실제 컴포넌트 렌더링이 올바르게 동작하는지를 순서대로 확인한다. JSDOM 환경에서 LitElement를 테스트하기 위해 [`./dom-setup.ts`](./dom-setup.ts.md)의 유틸리티를 사용한다.

## 의존성

### 외부 패키지
- `node:assert` — `assert` (Node.js 내장 어서션)
- `node:test` — `describe`, `it`, `beforeEach`, `after`, `before`
- `@a2ui/web_core/v0_9` — `MessageProcessor`

### 저장소 내부 모듈
- [`./dom-setup.js`](./dom-setup.ts.md) — `setupTestDom`, `teardownTestDom`, `asyncUpdate`
- `../surface/a2ui-surface.js` — `A2uiSurface` (타입 import, 동적 import로도 로드)
- `../catalogs/basic/index.js` — `basicCatalog` (동적 import)

## Exports

없음 (테스트 파일)

## 테스트 케이스

### describe: `A2uiSurface`

**픽스처 / 모킹**
- `before` 훅: `setupTestDom()`으로 JSDOM 전역을 주입한 뒤, `../surface/a2ui-surface.js`와 `../catalogs/basic/index.js`를 동적 import한다. 동적 import 순서가 중요한데, JSDOM 전역이 설정되기 전에 LitElement 모듈이 평가되면 Node.js 빈 컨텍스트에서 충돌이 발생하기 때문이다.
- `after` 훅: `teardownTestDom()`을 호출하여 DOM 전역을 복원한다.
- `beforeEach` 훅:
  1. `new MessageProcessor([basicCatalog])`로 프로세서를 생성한다.
  2. `version: 'v0.9'`, `createSurface: { surfaceId: 'test-surface', catalogId: basicCatalog.id }` 메시지를 처리하여 서피스를 초기화한다.
  3. `processor.model.getSurface('test-surface')`로 `surfaceModel`을 획득한다.

---

#### `should render nothing when surface is undefined`
- 검증 동작: `a2ui-surface` 엘리먼트를 생성하여 `surface` 속성을 설정하지 않고 DOM에 붙인 뒤, `el.shadowRoot?.innerHTML`이 정확히 `'<!---->'`인지 확인한다.
- 픽스처: `asyncUpdate`로 DOM에 append한 뒤 완료를 대기한다. 검증 후 `document.body.removeChild(el)`로 정리한다.

#### `should render loading state when surface has no root component`
- 검증 동작: `a2ui-surface` 엘리먼트에 `surface = surfaceModel`을 설정하되, 아직 루트 컴포넌트가 없는 상태에서 Shadow DOM에 `'Loading surface'` 문자열이 포함되어 있는지 확인한다.
- 픽스처: `asyncUpdate`로 `e.surface = surfaceModel` 설정 후 업데이트 완료 대기. 검증 후 `document.body.removeChild(el)`로 정리한다.

#### `should render root component once it becomes available`
- 검증 동작:
  1. 먼저 서피스를 설정하여 `'Loading surface'`가 렌더링됨을 중간 확인한다.
  2. 이후 `updateComponents` 메시지(`id: 'root'`, `component: 'Text'`, `text: 'Hello JSDOM'`)를 `processMessages`로 처리하여 루트 컴포넌트를 추가한다.
  3. `el.updateComplete`를 await하고, Shadow DOM에서 `a2ui-basic-text` 자식 엘리먼트를 찾아 그것의 `updateComplete`도 await한다.
  4. 최종적으로 `el.shadowRoot?.innerHTML`에 `'Loading surface'`가 없어야 하고, `childEl.shadowRoot?.innerHTML`에 `'Hello JSDOM'`이 포함되어야 함을 검증한다.
- 픽스처: `asyncUpdate`를 두 번 사용하여 각각 서피스 설정과 컴포넌트 추가 단계를 처리한다.

## 동작 흐름

테스트 스위트 시작 시 JSDOM 전역 설정 → Lit 컴포넌트 및 카탈로그 동적 로딩 순서로 초기화한다. 각 테스트에서 새로운 `MessageProcessor`로 독립적인 서피스 상태를 구성하고, DOM 엘리먼트를 직접 생성·조작하여 렌더링 결과를 Shadow DOM에서 검사한다.
