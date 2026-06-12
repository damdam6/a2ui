# renderers/react/tests/v0_8/unit/components/CheckBox.test.tsx

## 개요

A2UI v0.8 `CheckBox` 컴포넌트에 대한 단위 테스트 파일이다.
A2UI 스펙에 따라 `CheckBox`는 `label`과 `value` 두 필드를 필수로 받는다.
체크박스 입력 렌더링, 체크 상태 초기값, 레이블 연결(htmlFor/id 쌍), 사용자 인터랙션(클릭·change 이벤트), DOM 구조(section 컨테이너, input→label 순서)를 검증한다.

## 의존성

### 외부 패키지
- `vitest` — `describe`, `it`, `expect`
- `@testing-library/react` — `render`, `screen`, `fireEvent`
- `react` — `React`

### 저장소 내부 모듈
- [`../../utils`](../../utils/index.ts.md) — `TestWrapper`, `TestRenderer`, `createSimpleMessages`

## Exports

없음 (테스트 파일).

## 테스트 케이스 상세

### describe: `CheckBox Component`

모든 테스트의 공통 픽스처 패턴:
- `createSimpleMessages(id, 'CheckBox', {label: {literalString: '...'},  value: {literalBoolean: ...}})` 호출로 메시지를 생성한다.
- `<TestWrapper><TestRenderer messages={messages}/></TestWrapper>` 구조로 렌더링한다.

---

#### describe: `Basic Rendering`

- **`should render a checkbox input`**
  - `label: 'Accept terms'`, `value: false`.
  - `container.querySelector('input[type="checkbox"]')` 존재 확인.

- **`should render with wrapper div`**
  - `label: 'Subscribe'`, `value: false`.
  - `container.querySelector('.a2ui-checkbox')` 존재 확인 — 래퍼 div의 CSS 클래스가 `a2ui-checkbox`임을 보장.

- **`should render unchecked when value is false`**
  - `value: {literalBoolean: false}`.
  - `(checkbox as HTMLInputElement).checked === false` 확인.

- **`should render checked when value is true`**
  - `value: {literalBoolean: true}`.
  - `(checkbox as HTMLInputElement).checked === true` 확인.

- **`should render different states for different value inputs`**
  - `cb-1` (value: true)과 `cb-2` (value: false)를 각각 독립 렌더링.
  - `checkboxTrue.checked === true`, `checkboxFalse.checked === false` 확인, 두 값이 서로 다름을 교차 단언.

- **`should render different labels for different inputs`**
  - `cb-1` (label: `'First Option'`)과 `cb-2` (label: `'Second Option'`) 독립 렌더링.
  - `container.querySelector('label')?.textContent`가 각각의 레이블 문자열과 정확히 일치하며 서로 다름을 확인.

---

#### describe: `Label Rendering`

- **`should render label (required field)`**
  - `label: 'Accept terms and conditions'`.
  - `screen.getByText('Accept terms and conditions')` 존재 확인 — 레이블이 필수 필드임을 명시.

- **`should associate label with checkbox via htmlFor`**
  - `label: 'Remember me'`, `value: false`.
  - `label.getAttribute('for') === checkbox.id` 단언 — 레이블과 입력이 `htmlFor`/`id`로 올바르게 연결됨을 검증.

---

#### describe: `User Interaction`

- **`should toggle checked state on click`**
  - 초기 `value: false`.
  - `fireEvent.click(checkbox)` → `checked === true` → 다시 `fireEvent.click` → `checked === false` — 토글 동작 확인.

- **`should toggle via change event`**
  - `fireEvent.change(checkbox, {target: {checked: true}})` → `checked === true` 확인 — change 이벤트 경로로도 상태 변경 가능함을 검증.

---

#### describe: `Theme Support`

- **`should render within section container`**
  - `container.querySelector('section')` 존재 확인.
  - 그 `section` 안에 `input[type="checkbox"]`가 포함됨을 중첩 쿼리로 확인 — CheckBox가 section 컨테이너 내부에 렌더링됨을 보장.

---

#### describe: `Structure`

- **`should render checkbox before label (Lit structure)`**
  - `container.querySelector('section')?.children`의 첫 번째 자식 tagName이 `'INPUT'`, 두 번째가 `'LABEL'`임을 확인.
  - 즉, `<section><input type="checkbox" .../><label ...>...</label></section>` 순서임을 보장한다.

---

## 동작 흐름

모든 테스트는 동기적으로 실행된다(토글 상태 검사 포함). `createSimpleMessages`가 단일 `CheckBox` 컴포넌트 메시지를 생성하고, `TestRenderer`가 이를 처리해 DOM을 구성한다. 사용자 인터랙션 테스트에서 `fireEvent`는 React 합성 이벤트 시스템을 통해 체크박스 내부 상태를 변경하며, 상태 변화는 `checkbox.checked` 프로퍼티로 즉시 검증된다.
