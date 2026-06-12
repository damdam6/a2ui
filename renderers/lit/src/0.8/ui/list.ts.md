# renderers/lit/src/0.8/ui/list.ts

## 개요

`<a2ui-list>` Web Component를 정의한다. 자식 컴포넌트들을 수직(grid) 또는 수평(flex 스크롤) 방향으로 나열하는 레이아웃 컨테이너 역할을 한다. `direction` 속성이 호스트 엘리먼트에 reflect되어 CSS 속성 선택자로 레이아웃을 전환한다. 자식 콘텐츠는 `<slot>`을 통해 Light DOM에서 투영된다.

## 의존성

### 외부 패키지
- `lit` — `html`, `css`, `nothing`
- `lit/decorators.js` — `customElement`, `property`
- `lit/directives/class-map.js` — `classMap`
- `lit/directives/style-map.js` — `styleMap`

### 저장소 내부 모듈
- [`./root.js`](../root.ts.md) — 베이스 클래스 `Root`
- [`./styles.js`](../styles.ts.md) — `structuralStyles`

## Exports

| 이름 | 종류 | 설명 |
|---|---|---|
| `List` | 클래스 (LitElement 서브클래스) | `<a2ui-list>` 커스텀 엘리먼트 |

## 상세 명세

### `@customElement('a2ui-list') class List extends Root`

**Properties (Lit @property)**

- `accessor direction: 'vertical' | 'horizontal' = 'vertical'` — 데코레이터 옵션 `{ reflect: true, type: String }`. `reflect: true`이므로 DOM 속성으로도 반영되어 CSS `[direction='vertical']` / `[direction='horizontal']` 선택자가 동작한다.

**Static 필드**

- `static styles` — `structuralStyles`와 로컬 CSS 배열.
  - `:host`: `display: block`, `flex: var(--weight)`, `min-height: 0`, `overflow: auto`.
  - `:host([direction='vertical']) section`: `display: grid`.
  - `:host([direction='horizontal']) section`: `display: flex`, `max-width: 100%`, `overflow-x: scroll`, `overflow-y: hidden`, `scrollbar-width: none`. 슬롯 자식(`> ::slotted(*)`)에는 `flex: 1 0 fit-content`, `max-width: min(80%, 400px)`를 적용해 수평 스크롤 시 각 아이템의 너비를 제한한다.

---

#### `render(): TemplateResult`

1. `<section>` 엘리먼트를 반환한다.
2. `class`는 `classMap(this.theme.components.List)`.
3. `style`은 `this.theme.additionalStyles?.List`가 있으면 `styleMap(...)`, 없으면 `nothing`.
4. 내부에 `<slot></slot>`을 삽입해 자식 컴포넌트를 투영한다.

## 동작 흐름

`direction` 속성이 반영(reflect)되면 `:host([direction='...'])` CSS 선택자가 활성화되어 수직(grid)/수평(flex) 레이아웃이 전환된다. 자식 컴포넌트는 `Root.renderComponentTree`가 Light DOM에 렌더링하며, `<slot>`이 이를 Shadow DOM 내부로 투영한다. 테마 클래스와 추가 스타일은 `render` 시 `<section>`에 적용된다.
