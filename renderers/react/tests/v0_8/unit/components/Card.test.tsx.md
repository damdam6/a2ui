# renderers/react/tests/v0_8/unit/components/Card.test.tsx

## 개요

A2UI v0.8 `Card` 컴포넌트에 대한 단위 테스트 파일이다.
A2UI 스펙에 따라 `Card`는 `child` (자식 컴포넌트 ID 문자열) 하나를 필수로 받는다.
DOM 구조(wrapper div, section 태그), 자식 컴포넌트 렌더링, 테마 클래스 적용 여부를 검증한다.

## 의존성

### 외부 패키지
- `vitest` — `describe`, `it`, `expect`
- `@testing-library/react` — `render`, `screen`
- `react` — `React`
- `@a2ui/web_core/types/types` — `Types` 네임스페이스 (타입 전용)

### 저장소 내부 모듈
- [`../../utils`](../../utils/index.ts.md) — `TestWrapper`, `TestRenderer`, `createSurfaceUpdate`, `createBeginRendering`

## Exports

없음 (테스트 파일).

## 테스트 케이스 상세

### describe: `Card Component`

모든 테스트의 공통 픽스처 패턴:
- `createSurfaceUpdate`로 자식 컴포넌트(보통 `Text`)와 `Card` 컴포넌트를 동시에 등록한다.
- `createBeginRendering('card-1')`으로 `card-1`을 렌더링 루트로 지정한다.
- `<TestWrapper><TestRenderer messages={messages}/></TestWrapper>` 구조로 렌더링한다.

---

#### describe: `Basic Rendering`

- **`should render a section element`**
  - `text-1` (`Text`, literalString: `'Card content'`) + `card-1` (`Card`, child: `'text-1'`) 구성.
  - `container.querySelector('section')` 결과가 DOM에 존재함을 확인.

- **`should render with wrapper div`**
  - 동일한 구성.
  - `container.querySelector('.a2ui-card')`로 CSS 클래스 `a2ui-card`를 가진 래퍼 div 존재 확인.

---

#### describe: `Child Rendering`

- **`should render child Text component`**
  - `text-1` (`literalString: 'Card content'`) + `card-1` 구성.
  - `screen.getByText('Card content')` 존재 확인 — 자식 텍스트가 실제로 렌더링됨을 검증.

- **`should render nested Button in Card`**
  - `btn-text` (`Text`, `'Click me'`) → `btn-1` (`Button`, child: `'btn-text'`, action: `{name:'click'}`) → `card-1` (`Card`, child: `'btn-1'`) 3단계 중첩.
  - `screen.getByRole('button')` 존재 확인 및 `'Click me'` 텍스트 존재 확인 — Card가 Button 자식을 올바르게 전달함을 검증.

---

#### describe: `Theme Support`

- **`should apply theme classes to section`**
  - `text-1` + `card-1` 기본 구성.
  - `container.querySelector('section')`의 `className`이 `undefined`가 아닌 string 타입이며, `classList.length > 0`임을 확인 — 테마 클래스가 최소 1개 이상 적용됨을 검증.

---

#### describe: `Structure`

- **`should have correct DOM structure`**
  - `text-1` + `card-1` 기본 구성.
  - `.a2ui-card` 래퍼의 직계 자식이 정확히 1개이고, 그 자식의 `tagName`이 `'SECTION'`임을 확인.
  - 즉, `<div class="a2ui-card"><section>...</section></div>` 구조임을 보장한다.

---

## 동작 흐름

모든 테스트는 동기적으로 실행된다. `TestRenderer`가 메시지를 처리해 컴포넌트 트리를 구성하면, `Card`는 단일 자식 ID를 참조해 해당 컴포넌트를 내부에 렌더링한다. DOM 검사는 CSS 클래스 셀렉터(`'.a2ui-card'`)와 HTML 태그 이름(`'section'`, `'SECTION'`)을 기준으로 수행된다.
