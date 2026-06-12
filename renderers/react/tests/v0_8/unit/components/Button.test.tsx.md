# renderers/react/tests/v0_8/unit/components/Button.test.tsx

## 개요

A2UI v0.8 `Button` 컴포넌트에 대한 단위 테스트 파일이다.
A2UI 스펙에 따라 `Button`은 반드시 `child` (자식 컴포넌트 ID 문자열)와 `action.name`을 가져야 하며, `primary`는 선택적이다.
기본 렌더링 구조, 액션 디스패치 동작, 접근성을 검증한다.
로컬 헬퍼 `createButtonMessages`를 통해 메시지 픽스처를 일관되게 생성한다.

## 의존성

### 외부 패키지
- `vitest` — `describe`, `it`, `expect`, `vi`
- `@testing-library/react` — `render`, `screen`, `fireEvent`
- `react` — `React`
- `@a2ui/web_core/types/types` — `Types` 네임스페이스 (타입 전용)

### 저장소 내부 모듈
- [`../../utils`](../../utils/index.ts.md) — `TestWrapper`, `TestRenderer`, `createSurfaceUpdate`, `createBeginRendering`, `getMockCallArg`

## Exports

없음 (테스트 파일).

## 상세 명세

### `createButtonMessages(id, props, surfaceId?): Types.ServerToClientMessage[]`

**시그니처**:
- `id: string` — 버튼 컴포넌트 ID
- `props.actionName: string` — 버튼 클릭 시 발행할 액션 이름
- `props.actionContext?: Array<{key: string; value: {literalString?/literalNumber?/literalBoolean?/path?: ...}}>` — 선택적 컨텍스트 파라미터
- `props.childText?: string` — 버튼 내 텍스트 (기본값: `'Click me'`)
- `props.primary?: boolean` — primary 스타일 플래그 (선택)
- `surfaceId = '@default'` — 서피스 ID 기본값

**동작**:
1. `textId`를 `` `${id}-text` ``로 생성한다.
2. `createSurfaceUpdate`를 호출해 두 컴포넌트를 등록한다.
   - 먼저 `textId` ID를 가진 `Text` 컴포넌트: `text.literalString`은 `props.childText ?? 'Click me'`, `usageHint: 'body'`.
   - 그다음 `id` ID를 가진 `Button` 컴포넌트: `child: textId`, `action: {name: props.actionName, context: props.actionContext}`, `primary: props.primary`.
3. `createBeginRendering(id, surfaceId)`로 해당 컴포넌트를 루트로 렌더링 시작 메시지를 추가한다.
4. 두 메시지를 배열로 반환한다.

이 헬퍼는 A2UI 스펙 준수(child가 ID 참조)를 보장하며, 각 테스트에서 메시지 조립 반복을 제거한다.

---

## 테스트 케이스 상세

### describe: `Button Component`

#### describe: `Basic Rendering`

- **`should render a button element`**
  - `createButtonMessages('btn-1', {actionName:'submit'})` 사용.
  - `container.querySelector('button')`으로 `<button>` 태그 존재 및 `tagName === 'BUTTON'` 확인.

- **`should render with wrapper div having correct class`**
  - `container.querySelector('.a2ui-button')`으로 클래스 `a2ui-button`을 가진 `DIV` 래퍼 존재 확인.

- **`should render child text content inside button`**
  - `childText: 'Submit Form'` 지정.
  - `button.textContent`에 `'Submit Form'`이 포함됨을 확인. 텍스트가 버튼 바깥이 아닌 안쪽에 렌더링됨을 보장.

- **`should render different text for different child content`**
  - `btn-1`(`childText:'Save'`)과 `btn-2`(`childText:'Cancel'`) 두 개를 각각 독립 렌더링.
  - 각 버튼의 `textContent`가 서로 다름을 교차 검증.

---

#### describe: `Action Handling`

- **`should call onAction with correct action name when clicked`**
  - `mockOnAction = vi.fn()`, `actionName: 'submit-form'`.
  - 클릭 후 `mockOnAction` 1회 호출, `getMockCallArg<Types.A2UIClientEventMessage>(mockOnAction, 0)`로 첫 번째 인자를 추출.
  - `event.userAction.name === 'submit-form'` 확인 및 `'default'`, `'click'` 아님을 명시적 단언.

- **`should dispatch action with correct context parameters`**
  - `actionContext: [{key:'itemId', value:{literalString:'123'}}, {key:'confirmed', value:{literalBoolean:true}}]`.
  - 클릭 후 `event.userAction.name === 'delete-item'`, `event.userAction.context`가 객체 타입임을 확인.

- **`should not call onAction before click`**
  - 클릭 없이 렌더링만 수행 → `mockOnAction`이 호출되지 않음 확인.

- **`should call onAction multiple times for multiple clicks`**
  - 같은 버튼을 3회 `fireEvent.click` → `mockOnAction` 3회 호출 확인.

---

#### describe: `Accessibility`

- **`should be focusable`**
  - `button.focus()` 호출 후 `document.activeElement === button` 확인.

- **`should be clickable via role`**
  - `screen.getByRole('button')`으로 버튼 조회 — 적절한 ARIA role 보유 확인.
  - 클릭 후 `mockOnAction` 호출 확인.

---

## 동작 흐름

각 테스트는 `<TestWrapper>` (A2UIProvider 래핑) 안에 `<TestRenderer messages={messages} />`를 렌더링한다. `TestRenderer`는 메시지 배열을 받아 내부적으로 `processMessages`를 호출한다. 테스트는 렌더링된 DOM에서 `container.querySelector` 또는 `screen.*` API로 요소를 조회하고, 필요 시 `fireEvent`로 사용자 인터랙션을 시뮬레이션한다. 액션 검증은 `getMockCallArg` 헬퍼를 통해 `vi.fn()` 목 함수의 호출 인자를 타입 안전하게 추출한다.
