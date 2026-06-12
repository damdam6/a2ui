# renderers/lit/src/v0_9/catalogs/basic/components/Tabs.ts

## 개요

`a2ui-tabs` 커스텀 엘리먼트를 정의하는 파일이다. 탭 헤더 버튼과 선택된 탭의 콘텐츠를 렌더링하는 탭 UI 컴포넌트다. `@state`로 관리되는 `activeIndex`를 통해 현재 활성 탭을 추적하고, 헤더 버튼 클릭 시 해당 인덱스로 전환하여 대응하는 자식 노드를 렌더링한다.

## 의존성

### 외부 패키지
- `lit` — `html`, `nothing`, `css`
- `lit/decorators.js` — `customElement`, `state`
- `lit/directives/class-map.js` — `classMap`
- `@a2ui/web_core/v0_9/basic_catalog` — `TabsApi`
- `@a2ui/lit/v0_9` — `A2uiController`

### 저장소 내부 모듈
- [`../basic-catalog-a2ui-lit-element.ts`](../basic-catalog-a2ui-lit-element.ts.md) — `BasicCatalogA2uiLitElement`

## Exports

- `A2uiLitTabs` — 클래스 (커스텀 엘리먼트)
- `A2uiTabs` — 상수 (객체: `TabsApi` 스프레드 + `tagName: 'a2ui-tabs'`)

## 상세 명세

### `A2uiLitTabs` 클래스

**데코레이터**: `@customElement('a2ui-tabs')`

**상속**: `BasicCatalogA2uiLitElement<typeof TabsApi>`

#### 정적 필드

- `static styles`: `CSSResult`:
  - `:host`: `display: block`.
  - `.a2ui-tabs-headers`: `display: flex`, `gap: --a2ui-spacing-xs`, `border-bottom: --a2ui-tabs-border`(기본 `--a2ui-border-width` solid `--a2ui-color-border`), `margin-bottom: --a2ui-spacing-m`.
  - `.a2ui-tabs-header`: 헤더 버튼 기본 스타일. `padding: --a2ui-spacing-m --a2ui-spacing-l`, `background: --a2ui-tabs-header-background`(기본 `transparent`), `border: none`, `border-radius` 상단 두 모서리만 적용, `cursor: pointer`, `font-family: inherit`.
  - `.a2ui-tabs-header.active`: `background: --a2ui-tabs-header-background-active`(기본 `--a2ui-color-secondary`), `color: --a2ui-tabs-header-color-active`(기본 `--a2ui-color-on-secondary`).
  - `.a2ui-tabs-content`: `padding: --a2ui-tabs-content-padding`(기본 `0 --a2ui-spacing-m`).

#### 인스턴스 필드

- `@state() accessor activeIndex: number` — 기본값 `0`. 현재 활성 탭의 0-기반 인덱스. 변경 시 자동 재렌더링을 트리거한다.

#### `createController(): A2uiController`

`new A2uiController(this, TabsApi)`를 반환.

#### `render(): TemplateResult | typeof nothing`

1. `this.controller.props`가 없거나 `props.tabs`가 없으면 `nothing` 반환.
2. 탭 헤더 영역 `<div class="a2ui-tabs-headers">`:
   - `props.tabs.map((tab, i) => ...)` 으로 각 탭에 대해 `<button>` 렌더링.
   - 버튼 클래스: `'a2ui-tabs-header': true`, `'a2ui-tab-button': true`, `active: i === this.activeIndex`.
   - `@click`: `this.activeIndex = i` 로 activeIndex를 클릭된 탭의 인덱스로 업데이트.
   - 버튼 텍스트: `tab.title`.
3. 탭 콘텐츠 영역 `<div class="a2ui-tabs-content">`:
   - `props.tabs[this.activeIndex]`가 있으면 `this.renderNode(props.tabs[this.activeIndex].child)` 렌더링.
   - 없으면 `nothing`.

### `A2uiTabs` 상수

`TabsApi` 스프레드 + `tagName: 'a2ui-tabs'`.

## 동작 흐름

props에서 탭 배열 수신 → 모든 탭 제목을 헤더 버튼으로 렌더링(활성 탭은 `active` 클래스 적용) → `activeIndex`에 해당하는 탭의 `child` 노드만 콘텐츠 영역에 렌더링. 헤더 버튼 클릭 시 `activeIndex` 상태 변경 → Lit 반응성에 의해 자동 재렌더링으로 콘텐츠 전환.
