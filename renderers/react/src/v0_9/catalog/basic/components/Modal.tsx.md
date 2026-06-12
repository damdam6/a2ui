# renderers/react/src/v0_9/catalog/basic/components/Modal.tsx

## 개요

`Modal` 컴포넌트는 a2ui basic catalog의 `ModalApi` 스키마를 따르는 React 구현체다. 트리거 영역을 클릭하면 전체 화면 오버레이 위에 콘텐츠 패널을 표시하고, 오버레이 클릭 또는 닫기 버튼(×)으로 닫을 수 있다. 열림/닫힘 상태를 컴포넌트 로컬 `useState`로 관리하며, 스타일은 CSS 변수 기반이다.

## 의존성

### 외부 패키지
- `react` — `useState`
- `@a2ui/web_core/v0_9/basic_catalog` — `ModalApi`

### 저장소 내부 모듈
- [`../../../adapter`](../../../adapter.tsx.md) — `createComponentImplementation`
- [`../utils`](../utils.ts.md) — `useBasicCatalogStyles`

## Exports

| 이름 | 종류 |
|------|------|
| `Modal` | 상수 (ReactComponentImplementation) |

## 상세 명세

### `Modal`

`createComponentImplementation(ModalApi, renderFn)` 호출로 생성된 컴포넌트 구현체다.

**렌더 함수 시그니처:** `({props, buildChild}) => JSX.Element`

**매개변수:**
- `props` — `ModalApi` 스키마에서 추론된 props 객체
  - `props.trigger` — 모달을 여는 트리거 자식 컴포넌트 ID (옵셔널)
  - `props.content` — 모달 본문에 표시할 자식 컴포넌트 ID (옵셔널)
- `buildChild` — 자식 컴포넌트 ID를 React 노드로 변환하는 함수

**내부 상태:**
- `isOpen: boolean` — 초기값 `false`. 모달 표시 여부를 제어한다.

**동작 로직:**

1. `useBasicCatalogStyles()`를 호출하여 global CSS 변수를 DOM에 주입한다.
2. React Fragment를 반환하며, 두 개의 주요 블록으로 구성된다.

**트리거 영역:**
- `display: 'inline-block'` 스타일을 가진 `<div>`로 감싸며, 클릭 시 `setIsOpen(true)`를 호출한다.
- `props.trigger`가 존재하면 `buildChild(props.trigger)`로 트리거 자식을 렌더링하고, 없으면 `null`을 렌더링한다.

**오버레이 + 모달 패널 (`isOpen === true`일 때만 렌더링):**
- 오버레이 `<div>`: `position: fixed`, `top/left/right/bottom: 0`, `zIndex: 1000`, CSS 변수 `--a2ui-modal-overlay-color` (기본값 `rgba(0, 0, 0, 0.5)`)로 반투명 배경 처리. `display: flex`로 가운데 정렬. 클릭 시 `setIsOpen(false)` 호출.
- 내부 콘텐츠 `<div>`: `onClick={e => e.stopPropagation()}`으로 버블링 방지. CSS 변수 기반 `background-color`(`--a2ui-color-surface`, 기본 `#fff`), `padding`(`--a2ui-modal-padding` → `--a2ui-spacing-l` → `24px`), `border-radius`(`--a2ui-modal-border-radius` → `--a2ui-border-radius` → `8px`). `maxWidth: '90%'`, `maxHeight: '90%'`, `overflow: 'auto'`, `flexDirection: 'column'`.
- 닫기 버튼: 오른쪽 정렬 `<button>`, 클릭 시 `setIsOpen(false)`. `&times;` 문자(×) 표시. `border: none`, `background: none`, `fontSize` CSS 변수 `--a2ui-font-size-xl`(기본 `1.5rem`).
- 콘텐츠 영역: `flex: 1`인 `<div>` 안에 `props.content`가 있으면 `buildChild(props.content)`, 없으면 `null`.

## 동작 흐름

컴포넌트 마운트 시 `isOpen = false`. 트리거 영역 클릭 → `isOpen = true` → 오버레이와 모달 패널이 렌더링됨. 오버레이 클릭 또는 닫기 버튼 클릭 → `isOpen = false` → 오버레이와 패널이 DOM에서 제거됨. 내부 콘텐츠 클릭은 이벤트 전파가 차단되어 모달이 닫히지 않는다.
