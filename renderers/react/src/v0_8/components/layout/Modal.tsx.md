# renderers/react/src/v0_8/components/layout/Modal.tsx

## 개요

`Modal` 컴포넌트는 콘텐츠를 다이얼로그 오버레이로 표시하는 컨테이너다. 닫힌 상태에서는 진입점 자식(`entryPointChild`)을 렌더링하고, 열린 상태에서는 HTML5 네이티브 `<dialog>` 요소를 `showModal()`로 활성화하여 콘텐츠 자식(`contentChild`)을 표시한다. 포털(portal) 없이 `.a2ui-surface` DOM 트리 안에서 인플레이스로 렌더링하여 CSS 선택자가 올바르게 동작하도록 설계되었다.

## 의존성

### 외부 패키지
- `react`: `useState`, `useCallback`, `useRef`, `useEffect`, `memo`
- `@a2ui/web_core/types/types`: `Types.ModalNode` (타입 전용 import)

### 저장소 내부 모듈
- [`../../types`](../../types.ts.md): `A2UIComponentProps` 타입
- [`../../hooks/useA2UIComponent`](../../hooks/useA2UIComponent.ts.md): `useA2UIComponent` 훅
- [`../../lib/utils`](../../lib/utils.ts.md): `classMapToString`, `stylesToObject` 유틸
- [`../../core/ComponentNode`](../../core/ComponentNode.tsx.md): `ComponentNode` 재귀 렌더러

## Exports

- `Modal` — memo로 감싼 named export (React 함수 컴포넌트)
- `default` — `Modal`을 default export로도 제공

## 상세 명세

### `Modal` (컴포넌트)

**시그니처:** `memo(function Modal({ node, surfaceId }: A2UIComponentProps<Types.ModalNode>): JSX.Element)`

**상태 및 ref:**

- `isOpen: boolean` — 다이얼로그가 열린 상태인지 여부. 초기값 `false`.
- `dialogRef: React.RefObject<HTMLDialogElement>` — 네이티브 `<dialog>` 요소에 대한 ref.

**내부 훅 및 함수:**

1. `useA2UIComponent(node, surfaceId)`로 `theme`을 가져온다.
2. `useState(false)`로 `isOpen` 상태를 초기화한다.
3. `useRef<HTMLDialogElement>(null)`로 `dialogRef`를 생성한다.
4. `openModal` 콜백: 의존성 없이 `setIsOpen(true)`를 호출한다.
5. `closeModal` 콜백: 의존성 없이 `setIsOpen(false)`를 호출한다.

**useEffect — 다이얼로그 열기 및 native close 이벤트 동기화:**

의존성은 `[isOpen]`이다. `dialogRef.current`가 없으면 즉시 반환한다. `isOpen`이 `true`이고 `dialog.open`이 `false`이면 `dialog.showModal()`을 호출하여 네이티브 최상위 레이어 오버레이를 활성화한다. 네이티브 `close` 이벤트 리스너를 등록하여 Escape 키 등으로 브라우저가 직접 닫을 때 `setIsOpen(false)`를 호출해 React 상태를 동기화한다. cleanup 함수에서 해당 리스너를 제거한다.

**`handleBackdropClick` 콜백:**

`React.MouseEvent<HTMLDialogElement>`를 받아 `e.target === e.currentTarget`인 경우, 즉 `<dialog>` 배경 영역 직접 클릭 시에만 `closeModal()`을 호출한다. dialog 내부 콘텐츠 클릭은 버블링되어 target이 currentTarget과 달라지므로 무시된다. 의존성은 `[closeModal]`이다.

**`handleKeyDown` 콜백:**

`React.KeyboardEvent<HTMLDialogElement>`를 받아 `e.key === 'Escape'`이면 `closeModal()`을 호출한다. jsdom 테스트 환경 호환성을 위해 추가된 핸들러이며, 실제 브라우저에서는 네이티브 `<dialog>` 동작이 Escape를 처리한다. 의존성은 `[closeModal]`이다.

**hostStyle 계산:**

`node.weight`가 정의된 경우 `{'--weight': node.weight}` CSS 변수 객체를 생성한다.

**렌더링 — 닫힌 상태 (`!isOpen`):**

`<div className="a2ui-modal">`에 `hostStyle`을 적용한다. 내부에 `<section onClick={openModal} style={{cursor: 'pointer'}}>`를 렌더링하고, 그 안에 `<ComponentNode node={props.entryPointChild} surfaceId={surfaceId} />`를 배치한다. 섹션 전체가 클릭 가능하다.

**렌더링 — 열린 상태 (`isOpen`):**

`<div className="a2ui-modal">`에 `hostStyle`을 적용한다. 내부에 `<dialog ref={dialogRef}>`를 배치하며, `classMapToString(theme.components.Modal.backdrop)` 클래스, `onClick={handleBackdropClick}`, `onKeyDown={handleKeyDown}`을 적용한다. `<dialog>` 안에는 `<section>`(클래스: `theme.components.Modal.element`, 스타일: `stylesToObject(theme.additionalStyles?.Modal)`)이 있다. `<section>` 안에는 `<div id="controls">` 안에 닫기 버튼(`<button onClick={closeModal} aria-label="Close modal">`, 내부에 `<span className="g-icon">close</span>`)이 있고, 그 뒤에 `<ComponentNode node={props.contentChild} surfaceId={surfaceId} />`가 온다.

## 동작 흐름

초기에는 닫힌 상태로 `entryPointChild`만 표시된다. 사용자가 클릭하면 `openModal`이 `isOpen`을 `true`로 바꾸고, `useEffect`가 `showModal()`을 호출하여 네이티브 오버레이를 활성화한다. 모달을 닫는 방법은 세 가지다: 닫기 버튼 클릭(`closeModal`), 배경 영역 클릭(`handleBackdropClick`), Escape 키(`handleKeyDown` 또는 네이티브 `close` 이벤트 핸들러). 어떤 방법으로 닫혀도 `isOpen`이 `false`로 돌아가고 `entryPointChild` 표시로 복귀한다.
