# renderers/lit/src/v0_9/catalogs/basic/components/List.ts

## 개요

`a2ui-list` 커스텀 엘리먼트를 정의하는 파일이다. 자식 노드들을 수평 또는 수직 방향으로 나열하는 범용 리스트 레이아웃 컨테이너다. `ListApi`의 `direction` prop에 따라 `flex-direction`을 업데이트 후 동적으로 변경하며 `overflow: auto`로 스크롤을 지원한다.

## 의존성

### 외부 패키지
- `lit` — `html`, `nothing`, `css`, `PropertyValues`
- `lit/decorators.js` — `customElement`
- `lit/directives/map.js` — `map`
- `@a2ui/web_core/v0_9/basic_catalog` — `ListApi`
- `@a2ui/lit/v0_9` — `A2uiController`

### 저장소 내부 모듈
- [`../basic-catalog-a2ui-lit-element.ts`](../basic-catalog-a2ui-lit-element.ts.md) — `BasicCatalogA2uiLitElement`, `ResolvedChildList` (타입)

## Exports

- `A2uiListElement` — 클래스 (커스텀 엘리먼트)
- `A2uiList` — 상수 (객체: `ListApi` 스프레드 + `tagName: 'a2ui-list'`)

## 상세 명세

### `A2uiListElement` 클래스

**데코레이터**: `@customElement('a2ui-list')`

**상속**: `BasicCatalogA2uiLitElement<typeof ListApi>`

#### 정적 필드

- `static styles`: `CSSResult` — `:host`에 `display: flex`, `overflow: auto`, `gap: var(--a2ui-list-gap, var(--a2ui-spacing-m, 0.5rem))`, `padding: var(--a2ui-list-padding, 0)`. `flex-direction`은 인라인 스타일로 동적 관리된다.

#### `createController(): A2uiController`

`new A2uiController(this, ListApi)`를 반환.

#### `updated(changedProperties: PropertyValues): void`

1. `super.updated(changedProperties)` 먼저 호출.
2. `this.controller.props`가 있으면, `props.direction === 'horizontal'`이면 `this.style.flexDirection = 'row'`, 그 외이면 `'column'`으로 설정한다.

#### `render(): TemplateResult | typeof nothing`

1. `this.controller.props`가 없으면 `nothing` 반환.
2. `props.children`이 배열이면 그대로, 아니면 `[]`로 `children`을 설정한다 (`ResolvedChildList` 타입).
3. `map` 디렉티브로 `children`을 순회하며 각 `child`에 대해 `this.renderNode(child)` 호출.

### `A2uiList` 상수

`ListApi` 스프레드 + `tagName: 'a2ui-list'`.

## 동작 흐름

props 수신 → `render()`에서 자식 목록 렌더링 → `updated()`에서 `direction` prop을 인라인 `flex-direction` 스타일로 적용. `overflow: auto`로 컨텐츠가 많을 때 스크롤이 자동으로 활성화된다.
