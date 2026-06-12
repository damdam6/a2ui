# renderers/lit/src/v0_9/catalogs/basic/components/Row.ts

## 개요

`a2ui-basic-row` 커스텀 엘리먼트를 정의하는 파일이다. 자식 노드들을 수평 방향으로 배치하는 레이아웃 컨테이너다. `RowApi`의 `justify`와 `align` props를 받아 flexbox의 `justifyContent`와 `alignItems`를 동적으로 조정하며, `Column.ts`와 대칭적인 구조를 가진다.

## 의존성

### 외부 패키지
- `lit` — `html`, `nothing`, `css`, `PropertyValues`
- `lit/decorators.js` — `customElement`
- `lit/directives/map.js` — `map`
- `@a2ui/web_core/v0_9/basic_catalog` — `RowApi`
- `@a2ui/lit/v0_9` — `A2uiController`

### 저장소 내부 모듈
- [`../basic-catalog-a2ui-lit-element.ts`](../basic-catalog-a2ui-lit-element.ts.md) — `BasicCatalogA2uiLitElement`, `ResolvedChildList` (타입)

## Exports

- `A2uiBasicRowElement` — 클래스 (커스텀 엘리먼트)
- `A2uiRow` — 상수 (객체: `RowApi` 스프레드 + `tagName: 'a2ui-basic-row'`)

## 상세 명세

### 모듈 수준 상수

#### `JUSTIFY_MAP: Record<string, string>`

Column.ts의 동일 이름 상수와 동일한 매핑:
- `'start'` → `'flex-start'`, `'center'` → `'center'`, `'end'` → `'flex-end'`
- `'spaceBetween'` → `'space-between'`, `'spaceAround'` → `'space-around'`, `'spaceEvenly'` → `'space-evenly'`, `'stretch'` → `'stretch'`

#### `ALIGN_MAP: Record<string, string>`

Column.ts의 동일 이름 상수와 동일한 매핑:
- `'start'` → `'flex-start'`, `'center'` → `'center'`, `'end'` → `'flex-end'`, `'stretch'` → `'stretch'`

### `A2uiBasicRowElement` 클래스

**데코레이터**: `@customElement('a2ui-basic-row')`

**상속**: `BasicCatalogA2uiLitElement<typeof RowApi>`

#### 정적 필드

- `static styles`: `CSSResult` — `:host`에 `display: flex`, `flex-direction: row`, `gap: var(--a2ui-row-gap, var(--a2ui-spacing-m))`. Column과 달리 `flex-direction`이 `row`로 고정된다.

#### `createController(): A2uiController`

`new A2uiController(this, RowApi)`를 반환.

#### `updated(changedProperties: PropertyValues): void`

1. `super.updated(changedProperties)` 먼저 호출.
2. `this.controller.props`가 있으면:
   - `JUSTIFY_MAP[props.justify ?? '']` 결과가 있으면 해당 값, 없으면 `'flex-start'`를 `this.style.justifyContent`에 설정.
   - `ALIGN_MAP[props.align ?? '']` 결과가 있으면 해당 값, 없으면 `'stretch'`를 `this.style.alignItems`에 설정.

#### `render(): TemplateResult | typeof nothing`

1. `this.controller.props`가 없으면 `nothing` 반환.
2. `props.children`이 배열이면 그대로, 아니면 `[]`로 `children` 설정 (`ResolvedChildList` 타입).
3. `map` 디렉티브로 `children`을 순회하며 `this.renderNode(child)` 호출.

### `A2uiRow` 상수

`RowApi` 스프레드 + `tagName: 'a2ui-basic-row'`.

## 동작 흐름

Column.ts와 동일한 패턴으로 동작하되 `flex-direction`이 `row`로 고정된다. props 수신 → `render()`에서 자식 목록 렌더링 → `updated()`에서 `justify`·`align`을 인라인 스타일로 변환 적용.
