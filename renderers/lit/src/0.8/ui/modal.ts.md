# renderers/lit/src/0.8/ui/modal.ts

## 개요

`<a2ui-modal>` Web Component를 정의한다. 엔트리포인트 슬롯(`slot="entry"`)을 클릭하면 네이티브 `<dialog>` 엘리먼트를 모달로 열고, 배경 클릭 또는 닫기 버튼으로 닫을 수 있다. 내부 상태 `#showModal`로 열림/닫힘을 전환하며, `ref` 디렉티브로 `<dialog>.showModal()`을 `requestAnimationFrame` 시점에 호출해 네이티브 모달을 활성화한다.

## 의존성

### 외부 패키지
- `lit` — `html`, `css`, `nothing`
- `lit/decorators.js` — `customElement`, `query`, `state`
- `lit/directives/class-map.js` — `classMap`
- `lit/directives/style-map.js` — `styleMap`
- `lit/directives/ref.js` — `ref`

### 저장소 내부 모듈
- [`./root.js`](../root.ts.md) — 베이스 클래스 `Root`
- [`./styles.js`](../styles.ts.md) — `structuralStyles`

## Exports

| 이름 | 종류 | 설명 |
|---|---|---|
| `Modal` | 클래스 (LitElement 서브클래스) | `<a2ui-modal>` 커스텀 엘리먼트 |

## 상세 명세

### `@customElement('a2ui-modal') class Modal extends Root`

**Static 필드**

- `static styles` — `structuralStyles`와 로컬 CSS 배열.
  - `dialog`: `padding: 0`, `border: none`, `background: none`.
  - `dialog section #controls`: `display: flex`, `justify-content: end`, `margin-bottom: 4px`.
  - `#controls button`: `padding: 0`, `background: none`, `width/height: 20px`, `border: none`, `cursor: pointer`.

**Private 상태 필드**

- `@state() accessor #showModal = false` — `true`이면 `<dialog>` 모달 렌더링, `false`이면 엔트리포인트 `<section>` 렌더링.

**Private 쿼리 필드**

- `@query('dialog') accessor #modalRef: HTMLDialogElement | null = null` — Shadow DOM 내 `<dialog>` 엘리먼트에 대한 참조.

---

#### `#closeModal(): void`

Private 메서드.
1. `#modalRef`가 null이면 즉시 반환.
2. `#modalRef.open`이 `true`이면 `#modalRef.close()`를 호출해 네이티브 dialog를 닫는다.
3. `#showModal = false`로 설정해 Lit 재렌더링을 트리거한다.

---

#### `render(): TemplateResult`

두 가지 뷰를 조건부로 반환한다.

**닫힘 상태 (`#showModal === false`):**
- `<section>` 하나를 반환한다.
- `@click` 핸들러: `this.#showModal = true`로 설정해 열림 상태로 전환한다.
- 내부에 `<slot name="entry"></slot>`을 삽입한다. 엔트리포인트 자식 컴포넌트(예: 버튼)가 이 슬롯에 투영된다.

**열림 상태 (`#showModal === true`):**
- `<dialog>` 엘리먼트를 반환한다.
  - `class`는 `classMap(this.theme.components.Modal.backdrop)`.
  - `@click` 핸들러: `evt.composedPath()`의 첫 번째 요소(`[top]`)가 `HTMLDialogElement` 인스턴스인 경우에만 (즉, dialog 배경 직접 클릭 시에만) `#closeModal()`을 호출한다. 자식 요소 클릭은 이벤트 버블링으로 도달하지만 `composedPath()[0]`이 dialog가 아니므로 무시된다.
  - `ref` 디렉티브 콜백: `el`이 `HTMLDialogElement` 인스턴스이고 아직 열려있지 않으면(`!el.open`) `requestAnimationFrame` 안에서 `el.showModal()`을 호출해 네이티브 모달을 활성화한다. `el`이 없거나 이미 열려있으면 아무것도 하지 않는다.
  - 내부 `<section>`:
    - `class`는 `classMap(this.theme.components.Modal.element)`.
    - `style`은 `this.theme.additionalStyles?.Modal`이 있으면 `styleMap(...)`, 없으면 `nothing`.
    - `<div id="controls">`: 닫기 버튼(`@click` → `#closeModal()`)과 Material Symbols `close` 아이콘 스팬을 포함.
    - `<slot></slot>`: 모달 콘텐츠 자식 컴포넌트가 투영된다.

## 동작 흐름

초기에는 엔트리포인트 섹션만 렌더링된다. 사용자가 클릭하면 `#showModal`이 `true`로 변경되어 재렌더링이 발생하고 `<dialog>`가 DOM에 추가된다. `ref` 콜백이 `requestAnimationFrame` 시점에 `showModal()`을 호출해 네이티브 모달을 열고 백드롭을 표시한다. 배경 클릭 또는 닫기 버튼 클릭 시 `#closeModal`이 dialog를 닫고 `#showModal`을 `false`로 되돌려 엔트리포인트 뷰로 복귀한다.
