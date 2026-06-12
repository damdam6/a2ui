# renderers/lit/src/v0_9/catalogs/basic/components/Modal.ts

## 개요

`a2ui-modal` 커스텀 엘리먼트를 정의하는 파일이다. 트리거 요소를 클릭하면 네이티브 HTML `<dialog>` 엘리먼트를 모달로 열고, 닫기 버튼으로 닫을 수 있는 모달 대화상자를 구현한다. `ModalApi`의 `trigger`와 `content` props를 각각 트리거 영역과 모달 내용으로 렌더링한다.

## 의존성

### 외부 패키지
- `lit` — `html`, `nothing`, `css`
- `lit/decorators.js` — `customElement`, `query`
- `@a2ui/web_core/v0_9/basic_catalog` — `ModalApi`
- `@a2ui/lit/v0_9` — `A2uiController`

### 저장소 내부 모듈
- [`../basic-catalog-a2ui-lit-element.ts`](../basic-catalog-a2ui-lit-element.ts.md) — `BasicCatalogA2uiLitElement`

## Exports

- `A2uiLitModal` — 클래스 (커스텀 엘리먼트)
- `A2uiModal` — 상수 (객체: `ModalApi` 스프레드 + `tagName: 'a2ui-modal'`)

## 상세 명세

### `A2uiLitModal` 클래스

**데코레이터**: `@customElement('a2ui-modal')`

**상속**: `BasicCatalogA2uiLitElement<typeof ModalApi>`

#### 정적 필드

- `static styles`: `CSSResult`:
  - `:host`: `display: inline-block`.
  - `dialog`: `border: 1px solid --a2ui-color-border`, `border-radius: --a2ui-modal-border-radius`(기본 `8px`), `padding: --a2ui-modal-padding`(기본 `24px`), `min-width: 300px`, `background: --a2ui-color-surface`.
  - `dialog::backdrop`: `background: --a2ui-modal-backdrop-bg`(기본 `rgba(0, 0, 0, 0.5)`).

#### 인스턴스 필드

- `@query('dialog') accessor dialog!: HTMLDialogElement` — Shadow DOM 내의 `<dialog>` 엘리먼트에 대한 참조. 렌더링 후 사용 가능하며 null이 아님을 `!`로 단언한다.

#### `createController(): A2uiController`

`new A2uiController(this, ModalApi)`를 반환.

#### `render(): TemplateResult | typeof nothing`

1. `this.controller.props`가 없으면 `nothing` 반환.
2. 다음 두 부분을 포함하는 템플릿을 반환한다:
   - **트리거 영역**: `<div @click={...} class="a2ui-modal-trigger" style="display: contents;">` — 클릭 시 `this.dialog?.showModal()`을 호출하여 모달을 연다. `props.trigger`가 있으면 `this.renderNode(props.trigger)`로 렌더링한다.
   - **대화상자**: `<dialog class="a2ui-modal a2ui-modal-overlay">` — 내부에 닫기 폼(`<form method="dialog">`)과 닫기 버튼(`<button class="a2ui-modal-close">×</button>`)을 포함한다. `<form method="dialog">` 내의 버튼 클릭은 브라우저 네이티브 동작으로 dialog를 닫는다. `props.content`가 있으면 `this.renderNode(props.content)`로 내용을 렌더링한다.

### `A2uiModal` 상수

`ModalApi` 스프레드 + `tagName: 'a2ui-modal'`.

## 동작 흐름

렌더링 시 트리거와 대화상자를 동시에 DOM에 삽입 → 트리거 `div` 클릭 시 `dialog.showModal()` 호출로 네이티브 모달 오버레이 활성화 → `<form method="dialog">` 내 버튼 클릭으로 브라우저 네이티브 닫기 동작 실행. `display: contents`를 사용하여 트리거 `div`가 레이아웃에 영향을 주지 않는다.
