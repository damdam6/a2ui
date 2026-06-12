# renderers/lit/src/v0_9/catalogs/basic/components/Column.ts

## 개요

`a2ui-basic-column` 커스텀 엘리먼트를 정의하는 파일이다. 자식 노드들을 세로 방향으로 배치하는 레이아웃 컨테이너 역할을 한다. `ColumnApi`의 `justify`와 `align` props를 받아 flexbox의 `justifyContent`와 `alignItems`를 동적으로 조정하며, 자식 노드 목록을 순서대로 렌더링한다.

## 의존성

### 외부 패키지
- `lit` — `html`, `nothing`, `css`, `PropertyValues`
- `lit/decorators.js` — `customElement`
- `lit/directives/map.js` — `map`
- `@a2ui/web_core/v0_9/basic_catalog` — `ColumnApi`
- `@a2ui/lit/v0_9` — `A2uiController`

### 저장소 내부 모듈
- [`../basic-catalog-a2ui-lit-element.ts`](../basic-catalog-a2ui-lit-element.ts.md) — `BasicCatalogA2uiLitElement`, `ResolvedChildList` (타입)

## Exports

- `A2uiBasicColumnElement` — 클래스 (커스텀 엘리먼트)
- `A2uiColumn` — 상수 (객체: `ColumnApi` 스프레드 + `tagName: 'a2ui-basic-column'`)

## 상세 명세

### 모듈 수준 상수

#### `JUSTIFY_MAP: Record<string, string>`

API의 justify 값을 CSS flexbox 값으로 변환하는 매핑 테이블:
- `'start'` → `'flex-start'`
- `'center'` → `'center'`
- `'end'` → `'flex-end'`
- `'spaceBetween'` → `'space-between'`
- `'spaceAround'` → `'space-around'`
- `'spaceEvenly'` → `'space-evenly'`
- `'stretch'` → `'stretch'`

#### `ALIGN_MAP: Record<string, string>`

API의 align 값을 CSS flexbox 값으로 변환하는 매핑 테이블:
- `'start'` → `'flex-start'`
- `'center'` → `'center'`
- `'end'` → `'flex-end'`
- `'stretch'` → `'stretch'`

### `A2uiBasicColumnElement` 클래스

**데코레이터**: `@customElement('a2ui-basic-column')`

**상속**: `BasicCatalogA2uiLitElement<typeof ColumnApi>`

#### 정적 필드

- `static styles`: `CSSResult` — `:host`에 `display: flex`, `flex-direction: column`, `gap: var(--a2ui-column-gap, var(--a2ui-spacing-m))`를 설정한다.

#### `createController(): A2uiController`

`new A2uiController(this, ColumnApi)`를 반환하는 팩토리 메서드.

#### `updated(changedProperties: PropertyValues): void`

1. `super.updated(changedProperties)`를 먼저 호출한다.
2. `this.controller.props`가 존재하면:
   - `JUSTIFY_MAP[props.justify ?? '']`의 결과가 있으면 해당 값을, 없으면 `'flex-start'`를 `this.style.justifyContent`에 적용한다.
   - `ALIGN_MAP[props.align ?? '']`의 결과가 있으면 해당 값을, 없으면 `'stretch'`를 `this.style.alignItems`에 적용한다.

렌더링 후 매 업데이트마다 인라인 스타일을 동적으로 변경하여 정렬 동작을 제어한다.

#### `render(): TemplateResult | typeof nothing`

1. `this.controller.props`가 없으면 `nothing`을 반환한다.
2. `props.children`이 배열이면 그대로, 아니면 빈 배열 `[]`로 `children`을 설정한다 (`ResolvedChildList` 타입).
3. `map` 디렉티브로 `children`을 순회하며 각 `child`에 대해 `this.renderNode(child)`를 호출하여 렌더링한다.

### `A2uiColumn` 상수

`ColumnApi`를 스프레드하고 `tagName: 'a2ui-basic-column'`을 추가한 객체.

## 동작 흐름

Column.ts와 동일한 패턴으로 동작하되 `flex-direction`이 `column`으로 고정된다. props 수신 → `render()`에서 자식 노드 목록 추출 및 순서대로 렌더링 → `updated()`에서 `justify`·`align` props를 인라인 스타일로 변환하여 flexbox 정렬 적용. 인라인 스타일 변경은 CSS 변수보다 우선하므로 매 업데이트 후 정렬이 확실히 반영된다.
