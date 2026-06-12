# renderers/react/tests/v0_8/unit/components/Row.test.tsx

## 개요

`Row` 컴포넌트의 A2UI 명세 준수 여부를 검증하는 단위 테스트 파일이다. `distribution`(가로 배치 방식)과 `alignment`(세로 정렬 방식) 속성, 자식 컴포넌트 렌더링, DOM 구조, 테마 클래스 적용을 총망라하여 테스트한다. 모든 테스트는 `TestWrapper` / `TestRenderer` 조합을 통해 서버-클라이언트 메시지 흐름을 시뮬레이션한다.

## 의존성

### 외부 패키지
- `vitest` — `describe`, `it`, `expect`
- `@testing-library/react` — `render`, `screen`
- `react` — `React`
- `@a2ui/web_core/types/types` — 타입 전용 import (`Types`)

### 저장소 내부 모듈
- [`../../utils`](../../utils/index.ts.md) — `TestWrapper`, `TestRenderer`, `createSurfaceUpdate`, `createBeginRendering`

## Exports

이 파일은 아무것도 export하지 않는다. Vitest 테스트 스위트 파일이다.

## 상세 명세 — 테스트 케이스

### 헬퍼 / 픽스처 패턴

모든 테스트는 동일한 구성 패턴을 사용한다.

1. `createSurfaceUpdate(components)` — 컴포넌트 목록(`id` + `component` 객체 배열)을 담은 `SurfaceUpdate` 메시지를 생성한다.
2. `createBeginRendering(rootId)` — 특정 컴포넌트 ID를 루트로 렌더링 시작 메시지를 생성한다.
3. 두 메시지를 배열로 묶어 `TestRenderer`에 전달한다.
4. `TestWrapper`로 감싸 Provider 컨텍스트를 공급한다.

Row 컴포넌트의 props 구조:
- `children.explicitList`: 자식 컴포넌트 ID 배열 (필수)
- `distribution`: `'start' | 'center' | 'end' | 'spaceBetween' | 'spaceAround' | 'spaceEvenly'` (선택, 기본값 `'start'`)
- `alignment`: `'start' | 'center' | 'end' | 'stretch'` (선택, 기본값 `'stretch'`)

---

### `describe('Basic Rendering')`

#### `should render a section element`
- `explicitList: ['text-1']`를 가진 Row를 렌더링한다.
- `container.querySelector('section')`가 DOM에 존재함을 검증한다.

#### `should render with wrapper div`
- 동일 구성으로 렌더링한다.
- `.a2ui-row` 클래스를 가진 래퍼 엘리먼트가 DOM에 존재함을 검증한다.

---

### `describe('Children Rendering')`

#### `should render child Text components`
- `text-1`("Item 1"), `text-2`("Item 2") 두 Text 컴포넌트를 자식으로 가진 Row를 렌더링한다.
- `screen.getByText('Item 1')`과 `screen.getByText('Item 2')` 모두 DOM에 존재함을 검증한다.

#### `should render empty row with empty explicitList`
- `explicitList: []`인 Row를 렌더링한다.
- `.a2ui-row` 래퍼가 DOM에 존재함을 검증한다(자식 없이도 구조 유지).

---

### `describe('Alignment')`

#### `should default to stretch alignment`
- `alignment` 미지정 Row를 렌더링한다.
- `.a2ui-row`의 `data-alignment` 속성값이 `'stretch'`임을 검증한다.

#### `should set data-alignment="${alignment}"` (파라미터화)
- `alignments = ['start', 'center', 'end', 'stretch']` 각각에 대해 `forEach`로 실행된다.
- 해당 `alignment` 값을 명시적으로 전달한 Row를 렌더링한다.
- `.a2ui-row`의 `data-alignment` 속성값이 전달한 값과 일치함을 검증한다.

---

### `describe('Distribution')`

#### `should default to start distribution`
- `distribution` 미지정 Row를 렌더링한다.
- `.a2ui-row`의 `data-distribution` 속성값이 `'start'`임을 검증한다.

#### `should set data-distribution="${distribution}"` (파라미터화)
- `distributions = ['start', 'center', 'end', 'spaceBetween', 'spaceAround', 'spaceEvenly']` 각각에 대해 `forEach`로 실행된다.
- 해당 `distribution` 값을 명시적으로 전달한 Row를 렌더링한다.
- `.a2ui-row`의 `data-distribution` 속성값이 전달한 값과 일치함을 검증한다.

---

### `describe('Nested Components')`

#### `should render multiple Buttons in Row`
- `btn1-text`("Button 1"), `btn2-text`("Button 2") 두 Text와 각각을 child로 가진 Button 컴포넌트(`btn-1`, `btn-2`)를 생성한다. Row는 `explicitList: ['btn-1', 'btn-2']`로 두 버튼을 자식으로 가진다.
- `screen.getAllByRole('button')`의 길이가 2이고, 두 텍스트 모두 DOM에 존재함을 검증한다.

---

### `describe('Theme Support')`

#### `should apply theme classes to section`
- Row를 렌더링한다.
- `section` 엘리먼트의 `className`이 정의(defined)되어 있고, 타입이 `'string'`이며, `classList.length`가 0보다 큼을 검증한다(최소 하나 이상의 테마 클래스가 적용됨).

---

### `describe('Differential Behavior')`

#### `should render different alignment for different alignment inputs`
- `alignment: 'start'`인 Row와 `alignment: 'end'`인 Row를 각각 별도로 렌더링한다.
- 첫 번째 Row의 `data-alignment`가 `'start'`, 두 번째가 `'end'`임을 검증하고, 두 값이 서로 다름을 추가로 검증한다.

#### `should render different distribution for different distribution inputs`
- `distribution: 'center'`인 Row와 `distribution: 'spaceBetween'`인 Row를 각각 별도로 렌더링한다.
- 첫 번째 Row의 `data-distribution`이 `'center'`, 두 번째가 `'spaceBetween'`임을 검증하고, 두 값이 서로 다름을 추가로 검증한다.

---

### `describe('Structure')`

#### `should have correct DOM structure`
- Row를 렌더링한다.
- `.a2ui-row`의 직계 자식 수가 1이고, 그 자식의 `tagName`이 `'SECTION'`임을 검증한다. 즉 DOM 구조는 `div.a2ui-row > section`이다.

## 동작 흐름

1. 각 테스트는 `createSurfaceUpdate`로 컴포넌트 맵과 `createBeginRendering`으로 루트 ID를 가진 메시지 배열을 구성한다.
2. `TestWrapper > TestRenderer` 트리가 해당 메시지를 처리해 A2UI 컴포넌트 트리를 React DOM으로 렌더링한다.
3. 테스트는 `container.querySelector` 또는 `screen` 쿼리로 DOM 상태를 검사한다.
4. `data-alignment` / `data-distribution` data 속성을 통해 Row가 props를 올바르게 DOM에 반영하는지 확인한다.
5. 파라미터화 테스트는 `as const` 튜플에 `forEach`를 걸어 각 값에 대한 독립 테스트 케이스를 생성한다.
