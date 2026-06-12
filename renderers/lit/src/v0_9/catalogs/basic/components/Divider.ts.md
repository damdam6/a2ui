# renderers/lit/src/v0_9/catalogs/basic/components/Divider.ts

## 개요

`a2ui-divider` 커스텀 엘리먼트를 정의하는 파일이다. 수평 또는 수직 방향의 구분선 UI를 렌더링한다. `DividerApi`의 `axis` prop에 따라 수직이면 `<div>`로, 수평이면 의미론적으로 적절한 `<hr>`로 렌더링한다.

## 의존성

### 외부 패키지
- `lit` — `html`, `nothing`, `css`
- `lit/decorators.js` — `customElement`
- `lit/directives/class-map.js` — `classMap`
- `@a2ui/web_core/v0_9/basic_catalog` — `DividerApi`
- `@a2ui/lit/v0_9` — `A2uiController`

### 저장소 내부 모듈
- [`../basic-catalog-a2ui-lit-element.ts`](../basic-catalog-a2ui-lit-element.ts.md) — `BasicCatalogA2uiLitElement`

## Exports

- `A2uiDividerElement` — 클래스 (커스텀 엘리먼트)
- `A2uiDivider` — 상수 (객체: `DividerApi` 스프레드 + `tagName: 'a2ui-divider'`)

## 상세 명세

### `A2uiDividerElement` 클래스

**데코레이터**: `@customElement('a2ui-divider')`

**상속**: `BasicCatalogA2uiLitElement<typeof DividerApi>`

#### 정적 필드

- `static styles`: `CSSResult`:
  - `:host`는 `display: block`, `align-self: stretch`.
  - `.a2ui-divider.horizontal`: `height: 0`, `border: 0`, `border-top`으로 선을 표현. 마진은 위아래로 `--a2ui-divider-spacing`, 너비는 `100%`.
  - `.a2ui-divider.vertical`: `width`는 `--a2ui-border-width`(기본 `1px`), `background-color`로 선 색상, `height: 100%`, 마진은 좌우로 `--a2ui-divider-spacing`.

#### `createController(): A2uiController`

`new A2uiController(this, DividerApi)`를 반환.

#### `render(): TemplateResult | typeof nothing`

1. `this.controller.props`가 없으면 `nothing` 반환.
2. `classes` 객체 구성:
   - `'a2ui-divider': true` 항상 포함.
   - `vertical: props.axis === 'vertical'`
   - `horizontal: props.axis !== 'vertical'`
3. `props.axis === 'vertical'`이면 `<div class={classes}></div>` 반환.
4. 그 외이면 `<hr class={classes} />` 반환.

### `A2uiDivider` 상수

`DividerApi` 스프레드 + `tagName: 'a2ui-divider'`.

## 동작 흐름

props 수신 → `axis` 값으로 수직/수평 구분 → 클래스 맵 구성 → 수직이면 `<div>`, 수평이면 `<hr>`로 렌더링. 시각적 스타일은 CSS 변수로 외부에서 커스터마이징 가능하다.
