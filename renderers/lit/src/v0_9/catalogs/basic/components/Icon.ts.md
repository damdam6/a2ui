# renderers/lit/src/v0_9/catalogs/basic/components/Icon.ts

## 개요

`a2ui-icon` 커스텀 엘리먼트를 정의하는 파일이다. Material Symbols Outlined 폰트 기반의 아이콘 또는 SVG path 기반의 아이콘을 렌더링한다. camelCase 아이콘 이름을 Material Symbols의 snake_case 이름으로 변환하는 로직과, 일부 특수 이름에 대한 오버라이드 테이블을 포함한다.

## 의존성

### 외부 패키지
- `lit` — `html`, `nothing`, `css`
- `lit/decorators.js` — `customElement`
- `@a2ui/web_core/v0_9/basic_catalog` — `IconApi`
- `@a2ui/lit/v0_9` — `A2uiController`

### 저장소 내부 모듈
- [`../basic-catalog-a2ui-lit-element.ts`](../basic-catalog-a2ui-lit-element.ts.md) — `BasicCatalogA2uiLitElement`

## Exports

- `A2uiIconElement` — 클래스 (커스텀 엘리먼트)
- `A2uiIcon` — 상수 (객체: `IconApi` 스프레드 + `tagName: 'a2ui-icon'`)

## 상세 명세

### `ICON_NAME_OVERRIDES: Record<string, string>`

camelCase 이름을 Material Symbols의 실제 심볼 이름으로 수동 매핑하는 테이블:
- `'play'` → `'play_arrow'`
- `'rewind'` → `'fast_rewind'`
- `'favoriteOff'` → `'favorite_border'`
- `'starOff'` → `'star_border'`

### `toMaterialSymbol(name: string): string`

**매개변수**: `name: string`

**반환**: `string` — Material Symbols에서 사용하는 심볼 이름.

**동작**:
1. `ICON_NAME_OVERRIDES[name]`이 존재하면 해당 값을 즉시 반환한다.
2. 그 외에는 `name.replace(/[A-Z]/g, letter => '_' + letter.toLowerCase())`를 적용하여 camelCase를 snake_case로 변환한다. 예: `'arrowBack'` → `'arrow_back'`.

### `A2uiIconElement` 클래스

**데코레이터**: `@customElement('a2ui-icon')`

**상속**: `BasicCatalogA2uiLitElement<typeof IconApi>`

#### 정적 필드

- `static styles`: `CSSResult`:
  - `:where(:host)`에서 `--_icon-size`를 `--a2ui-icon-size` → `--a2ui-font-size-xl` → `24px` 순으로 폴백하는 내부 변수로 설정한다.
  - `:host`는 `display: inline-flex`, `align-items: center`, `justify-content: center`.
  - `.material-symbol`: Material Symbols Outlined 폰트 패밀리, `font-size: var(--_icon-size)`, 색상은 `--a2ui-icon-color`, `font-variation-settings: var(--a2ui-icon-font-variation-settings, 'FILL' 1)` (기본값으로 채워진 아이콘 사용).
  - `.svg`: `fill: currentColor`, `width`와 `height` 모두 `--_icon-size`.

#### `createController(): A2uiController`

`new A2uiController(this, IconApi)`를 반환.

#### `render(): TemplateResult | typeof nothing`

1. `this.controller.props`가 없으면 `nothing` 반환.
2. `name = props.name` 추출.
3. `isPath` 판단: `name`이 `null`이 아닌 객체이고 `'svgPath'` 키를 가지면 `true`.
4. `isPath`가 `true`이면: `(name as {svgPath: string}).svgPath`를 꺼내어 `<svg class="svg" viewBox="0 0 24 24"><path d={path}></path></svg>`를 반환한다.
5. `isPath`가 `false`이면: `name`이 문자열이면 `toMaterialSymbol(name)`, 아니면 빈 문자열 `''`을 `iconName`으로 설정하고 `<span class="material-symbol">{iconName}</span>`을 반환한다.

### `A2uiIcon` 상수

`IconApi` 스프레드 + `tagName: 'a2ui-icon'`.

## 동작 흐름

props에서 아이콘 이름 수신 → SVG path 객체인지 문자열 이름인지 분기 → SVG path이면 인라인 SVG로, 문자열이면 `toMaterialSymbol`로 변환 후 Material Symbols 폰트 span으로 렌더링.
