# renderers/react/tests/v0_8/unit/components/Modal.test.tsx

## 개요

A2UI 명세에 따라 `Modal` 컴포넌트의 렌더링·열기·닫기·접근성·DOM 구조를 검증하는 단위 테스트 파일이다. `Modal`은 `entryPointChild`(트리거 컴포넌트 ID)와 `contentChild`(본문 컴포넌트 ID)를 필수 프로퍼티로 요구한다. jsdom이 `HTMLDialogElement.showModal()`·`close()`를 지원하지 않으므로 `beforeEach`에서 `vi.fn()` mock으로 대체하고, `afterEach`에서 `vi.restoreAllMocks()`로 복원한다. 비동기 상태 변경은 `waitFor`로 처리한다.

## 의존성

### 외부 패키지
- `vitest` — `describe`, `it`, `expect`, `vi`, `beforeEach`, `afterEach`
- `@testing-library/react` — `render`, `screen`, `fireEvent`, `waitFor`
- `react` — `React`
- `@a2ui/web_core/types/types` — 타입 전용 import (`Types.ServerToClientMessage`, `Types.A2UIClientEventMessage`)

### 저장소 내부 모듈
- [`../../utils`](../../utils/index.ts.md) — `TestWrapper`, `TestRenderer`, `createSurfaceUpdate`, `createBeginRendering`, `getMockCallArg`

## Exports

이 파일은 아무것도 export하지 않는다.

## 테스트 케이스 명세

### 헬퍼 함수: `createModalMessages`

**시그니처:**
```
function createModalMessages(
  id: string,
  props: { triggerText: string; contentText: string },
  surfaceId = '@default',
): Types.ServerToClientMessage[]
```

**동작 로직:**
1. `triggerId = \`${id}-trigger\``, `contentId = \`${id}-content\``를 파생한다.
2. `createSurfaceUpdate`로 세 컴포넌트를 한 번에 등록한다: `Text` 트리거(`triggerId`), `Text` 본문(`contentId`), 이 둘을 각각 `entryPointChild`·`contentChild`로 가리키는 `Modal`(`id`).
3. `createBeginRendering(id, surfaceId)`를 두 번째 메시지로 추가해 2-요소 배열을 반환한다.

---

### 전역 테스트 훅

**`beforeEach`** — `HTMLDialogElement.prototype.showModal`을 `vi.fn()`으로 교체하되, 함수 본문에서 `this.setAttribute('open', '')`를 실행해 열림 상태를 시뮬레이션한다. `HTMLDialogElement.prototype.close`도 동일한 방식으로 교체하며, `this.removeAttribute('open')`를 실행한다.

**`afterEach`** — `vi.restoreAllMocks()`로 모든 mock을 원래 구현으로 복원한다.

---

### describe: `Modal Component`

#### describe: `Basic Rendering`

| 테스트명 | 검증 동작 | 픽스처/모킹 |
|---|---|---|
| `should render modal wrapper with correct class` | `.a2ui-modal` 요소가 DOM에 존재하고 `tagName`이 `'DIV'`이다 | `createModalMessages` — `triggerText: 'Open Modal'` |
| `should render entry point child (trigger) visible initially` | `'Click to Open'` 텍스트가 렌더 직후 화면에 표시된다 | `triggerText: 'Click to Open'` |
| `should NOT render modal content initially (before open)` | `'Secret Modal Content'`가 초기 상태에서 DOM에 없다 | `contentText: 'Secret Modal Content'` |
| `should render different trigger text for different inputs` | 두 인스턴스의 텍스트가 각각 `'Trigger A'`·`'Trigger B'`를 포함하고 서로 다르다 | 두 개의 독립 `render` 호출 |

#### describe: `Opening Modal`

| 테스트명 | 검증 동작 | 픽스처/모킹 |
|---|---|---|
| `should open modal when clicking entry point` | 트리거 클릭 후 `contentText`가 DOM에 나타난다 | `fireEvent.click` → `waitFor` |
| `should render dialog element when open` | 트리거 클릭 후 `dialog` 요소가 DOM에 존재한다 | `fireEvent.click` → `waitFor` |
| `should call showModal() on the dialog element` | `HTMLDialogElement.prototype.showModal` mock이 호출된다 | `vi.fn()` mock |
| `should render modal in portal (document.body)` | `document.body.querySelector('dialog')`로 portal 렌더링을 확인한다 | `document.body` 레벨 검사 |

#### describe: `Closing Modal`

| 테스트명 | 검증 동작 | 픽스처/모킹 |
|---|---|---|
| `should render close button inside modal` | 모달 열기 후 `role='button'`, `name=/close/i`인 버튼이 DOM에 존재한다 | `waitFor` + `screen.getByRole` |
| `should close modal when clicking close button` | 닫기 버튼 클릭 시 `contentText`가 DOM에서 사라진다 | `fireEvent.click` on close button |
| `should close modal when clicking backdrop (dialog element)` | `dialog` 요소 자체 클릭 시 모달이 닫힌다 | `fireEvent.click(dialog)` |
| `should NOT close when clicking modal content` | 본문 텍스트 클릭 후 모달이 열린 상태를 유지한다 | `fireEvent.click` on content text |
| `should close modal on Escape key` | `dialog`에 `{key: 'Escape'}` keyDown 이벤트 발생 시 모달이 닫힌다 | `fireEvent.keyDown(dialog, {key: 'Escape'})` |

#### describe: `Nested Content`

| 테스트명 | 검증 동작 | 픽스처/모킹 |
|---|---|---|
| `should render complex content inside modal` | `Column` + `Text`(usageHint `'h2'`) + `Button`으로 구성된 중첩 콘텐츠가 모달 내부에 렌더링된다. `Modal Title` 텍스트와 `name='Submit'` 버튼이 모두 존재한다 | 직접 조립한 복합 메시지 배열 |
| `should dispatch actions from buttons inside modal` | 모달 내 `Button` 클릭 시 `onAction` 콜백이 호출되고, `getMockCallArg`로 추출한 `event.userAction.name`이 `'modal-action'`이다 | `vi.fn()` mockOnAction, `getMockCallArg<Types.A2UIClientEventMessage>` |

#### describe: `Entry Point Styles`

| 테스트명 | 검증 동작 | 픽스처/모킹 |
|---|---|---|
| `should have cursor: pointer on entry point` | `.a2ui-modal > section`에 `toHaveStyle({cursor: 'pointer'})`가 통과한다 | 동기 렌더 |

#### describe: `Accessibility`

| 테스트명 | 검증 동작 | 픽스처/모킹 |
|---|---|---|
| `should have aria-label on close button` | 모달 열기 후 `document.querySelector('#controls button')`의 `aria-label` 속성이 `'Close modal'`이다 | `waitFor` |
| `should use native dialog element for accessibility` | 모달 열기 후 `dialog` 요소가 존재하고 `tagName`이 `'DIALOG'`이다 | `waitFor` |

#### describe: `Structure`

| 테스트명 | 검증 동작 | 픽스처/모킹 |
|---|---|---|
| `should have correct DOM structure when closed` | 닫힌 상태: `.a2ui-modal(DIV)` 안에 `.a2ui-text`가 있고, `dialog`는 DOM에 없다 | 동기 렌더 |
| `should have correct DOM structure when open` | 열린 상태: `dialog > section > #controls > button` 계층 구조가 모두 존재한다 | `waitFor` |

#### describe: `Re-opening Modal`

| 테스트명 | 검증 동작 | 픽스처/모킹 |
|---|---|---|
| `should be able to open modal after closing` | 열기 → 닫기 → 재열기 세 단계를 순서대로 수행해 마지막 단계에서 `contentText`가 다시 화면에 나타난다 | 순차 `fireEvent` + `waitFor` |

## 동작 흐름

1. `beforeEach`에서 jsdom 환경에 없는 `showModal`·`close`를 `vi.fn()`으로 주입해 `open` 속성을 직접 토글한다.
2. 각 테스트에서 `createModalMessages`(또는 직접 조립)로 메시지를 생성하고 `<TestWrapper>`로 렌더링한다.
3. 초기 상태 테스트는 동기적으로 `screen.queryBy*` / `container.querySelector`로 검증한다.
4. 상호작용 테스트는 `fireEvent.click`으로 사용자 이벤트를 시뮬레이션한 뒤, 비동기 DOM 변경을 `waitFor`로 기다려 검증한다.
5. 액션 테스트는 `getMockCallArg<Types.A2UIClientEventMessage>(mockOnAction, 0)`로 mock 콜백 첫 번째 인자를 추출해 `userAction.name`을 단언한다.
6. `afterEach`에서 모든 mock을 복원한다.
