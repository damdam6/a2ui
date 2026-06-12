# renderers/react/tests/v0_8/unit/components/Tabs.test.tsx

## 개요

`Tabs` 컴포넌트의 A2UI 명세 준수 여부를 검증하는 단위 테스트 파일이다. 탭 버튼 렌더링, 기본 선택 상태, 탭 전환 시 콘텐츠 교체 및 버튼 disabled 상태 변경, 중첩 컴포넌트 지원, 빈 `tabItems` 경계 케이스, 접근성(포커스/키보드 활성화)을 포괄적으로 검증한다. 파일 내부에 `createTabsMessages` 헬퍼를 정의하여 반복 픽스처 구성을 단순화한다.

## 의존성

### 외부 패키지
- `vitest` — `describe`, `it`, `expect`
- `@testing-library/react` — `render`, `screen`, `fireEvent`
- `react` — `React`
- `@a2ui/web_core/types/types` — 타입 전용 import (`Types`)

### 저장소 내부 모듈
- [`../../utils`](../../utils/index.ts.md) — `TestWrapper`, `TestRenderer`, `createSurfaceUpdate`, `createBeginRendering`

## Exports

이 파일은 아무것도 export하지 않는다. Vitest 테스트 스위트 파일이다.

## 상세 명세

### 헬퍼 함수

#### `createTabsMessages`

시그니처:
```
function createTabsMessages(
  id: string,
  props: { tabs: Array<{ title: string; contentText: string }> },
  surfaceId = '@default',
): Types.ServerToClientMessage[]
```

- `props.tabs` 배열을 순회하며 각 탭에 대해 `{id}-content-{index}` 형식의 Text 컴포넌트를 생성한다. `title`은 `{ literalString: tab.title }` 형태로, `child`는 해당 content ID 문자열로 매핑된다.
- 생성된 tabItems 배열을 가진 Tabs 컴포넌트를 `id` 키로 추가한다.
- `createSurfaceUpdate(components, surfaceId)`와 `createBeginRendering(id, surfaceId)` 두 메시지를 배열로 반환한다.
- `surfaceId` 기본값은 `'@default'`이다.

---

### 테스트 케이스

#### `describe('Basic Rendering')`

##### `should render a tabs container with correct class`
- 2개 탭을 가진 Tabs를 렌더링한다.
- `.a2ui-tabs` 클래스를 가진 `DIV` 엘리먼트가 DOM에 존재함을 검증한다.

##### `should render section element inside wrapper`
- 1개 탭으로 렌더링한다.
- `.a2ui-tabs > section` CSS 셀렉터로 선택되는 엘리먼트가 DOM에 존재함을 검증한다.

##### `should render buttons container with id="buttons"`
- 2개 탭으로 렌더링한다.
- `#buttons` ID를 가진 `DIV` 엘리먼트가 DOM에 존재함을 검증한다.

---

#### `describe('Tab Buttons')`

##### `should render a button for each tab`
- 3개 탭("First", "Second", "Third")으로 렌더링한다.
- `#buttons button` 셀렉터로 선택된 버튼 수가 정확히 3임을 검증한다.

##### `should display tab titles in buttons`
- "Overview", "Details" 2개 탭으로 렌더링한다.
- `screen.getByRole('button', { name: 'Overview' })`와 `{ name: 'Details' }` 모두 DOM에 존재함을 검증한다.

##### `should disable the currently selected tab button`
- "Tab 1", "Tab 2" 2개 탭으로 렌더링한다.
- 초기 상태에서 Tab 1 버튼이 `disabled`이고 Tab 2 버튼이 활성화되어 있음을 검증한다(첫 번째 탭이 기본 선택).

##### `should have different disabled states for different tabs`
- "Tab A", "Tab B" 2개 탭으로 렌더링한다.
- `tabA.disabled`와 `tabB.disabled`가 서로 다른 값(한쪽은 true, 다른 쪽은 false)임을 `not.toBe`로 검증한다.

---

#### `describe('Tab Selection')`

##### `should select first tab by default`
- "Tab 1", "Tab 2" 탭으로 렌더링한다.
- 첫 번째 탭의 `contentText`인 `'First tab content'`가 DOM에 존재함을 검증한다.

##### `should switch content when clicking a different tab`
- 초기 렌더링 후 `fireEvent.click(screen.getByRole('button', { name: 'Tab 2' }))` 클릭 이벤트를 발행한다.
- 클릭 후 `'Second tab content'`가 DOM에 존재함을 검증한다.

##### `should hide previous tab content when switching`
- Tab 2를 클릭한다.
- `screen.queryByText('First tab content')`가 DOM에 존재하지 않음을 검증한다(이전 탭 콘텐츠 언마운트).

##### `should update disabled state when switching tabs`
- 초기 상태에서 Tab 1 disabled / Tab 2 활성을 검증한다.
- Tab 2를 클릭한 후 Tab 1이 활성 / Tab 2가 disabled로 전환됨을 검증한다.

##### `should handle switching between multiple tabs`
- 3개 탭("Tab 1", "Tab 2", "Tab 3")으로 렌더링한다.
- Tab 3 클릭 → `'Content Three'` 표시 + `'Content One'` 비표시를 검증한다.
- Tab 2 클릭 → `'Content Two'` 표시 + `'Content Three'` 비표시를 검증한다.
- Tab 1 클릭 → `'Content One'` 표시를 검증한다. 전방위 탭 순환을 확인한다.

---

#### `describe('Nested Content')`

##### `should render complex nested content in tabs`
- 직접 `createSurfaceUpdate`/`createBeginRendering`을 사용해 복잡한 구조를 구성한다:
  - Tab "List": `text-1a`("Item 1"), `text-1b`("Item 2")를 자식으로 가진 `Column` 컴포넌트.
  - Tab "Action": "Click me" 텍스트를 child로 가진 `Button` 컴포넌트.
- 초기 상태에서 "Item 1", "Item 2" 텍스트가 DOM에 존재함을 검증한다.
- "Action" 탭을 클릭하면 `role='button'`이고 name이 `'Click me'`인 버튼이 DOM에 존재함을 검증한다.

---

#### `describe('Empty/Edge Cases')`

##### `should render with single tab`
- 1개 탭("Only Tab", "Only content")으로 렌더링한다.
- `#buttons button` 수가 1이고 `'Only content'` 텍스트가 DOM에 존재함을 검증한다.

##### `should handle empty tabItems gracefully`
- 직접 `createSurfaceUpdate`로 `Tabs: { tabItems: [] }`를 구성해 렌더링한다.
- `.a2ui-tabs` 래퍼는 존재하고, `#buttons button` 수가 0임을 검증한다(빈 배열 허용).

---

#### `describe('Theme Support')`

##### `should render section container (theme classes empty by design)`
- `litTheme.components.Tabs.container`가 의도적으로 비어 있음을 주석으로 명시한다. Tabs 스타일링은 구조적 CSS에서 담당한다.
- `.a2ui-tabs > section`이 DOM에 존재함만 검증한다.

##### `should render buttons container (theme classes empty by design)`
- `litTheme.components.Tabs.element`가 의도적으로 비어 있음을 주석으로 명시한다.
- `#buttons` 컨테이너가 DOM에 존재함만 검증한다.

##### `should render tab buttons (theme classes empty by design)`
- `litTheme.components.Tabs.controls.all/selected`가 의도적으로 비어 있음을 주석으로 명시한다.
- `#buttons button` 2개가 모두 DOM에 존재함을 검증한다.

---

#### `describe('Structure')`

##### `should have correct DOM structure: div.a2ui-tabs > section > #buttons + content`
- `.a2ui-tabs`가 `DIV`이고, 그 안의 `section`이 존재하며, `section` 내부에 `#buttons` 컨테이너와 `.a2ui-text` 텍스트 래퍼가 모두 존재함을 검증한다. 전체 DOM 트리 구조를 단일 테스트로 확인한다.

---

#### `describe('Accessibility')`

##### `should have focusable tab buttons`
- Tab 2 버튼에 `.focus()`를 호출한다.
- `document.activeElement`가 Tab 2 버튼과 동일함을 검증한다.

##### `should be keyboard activatable`
- Tab 2 버튼에 포커스 후 `fireEvent.keyDown(tab2Button, { key: 'Enter' })`와 `fireEvent.click(tab2Button)`을 순서대로 발행한다.
- 주석에 "Buttons respond to click, not keyDown by default"라고 명시되어 있어, keyDown 자체보다 click 이벤트가 동작의 핵심이다.
- `'Content 2'`가 DOM에 존재함을 검증한다.

## 동작 흐름

1. `createTabsMessages` 헬퍼 또는 직접 `createSurfaceUpdate`/`createBeginRendering` 호출로 메시지 배열을 구성한다.
2. `TestWrapper > TestRenderer`가 메시지를 처리해 Tabs 컴포넌트를 React DOM으로 렌더링한다.
3. DOM 구조: `div.a2ui-tabs > section > div#buttons + <현재탭콘텐츠>`.
4. 탭 전환 시 `fireEvent.click`으로 버튼을 클릭하면 선택된 탭 인덱스가 변경되고, 해당 콘텐츠 컴포넌트만 렌더링되며 나머지는 DOM에서 제거된다.
5. 현재 선택된 탭의 버튼은 `disabled=true`로 설정되고, 나머지 버튼은 활성 상태를 유지한다.
6. 테마 관련 테스트는 `litTheme` 적용 여부가 아닌 구조적 DOM 존재 여부만 확인한다(테마 클래스가 의도적으로 비어 있기 때문).
