# renderers/lit/src/v0_9/catalogs/basic/components/Image.ts

## 개요

`a2ui-image` 커스텀 엘리먼트를 정의하는 파일이다. URL 기반 이미지를 렌더링하며 `icon`, `avatar`, `smallFeature`, `largeFeature`, `header` 등 여러 variant를 지원한다. `fit` prop으로 `object-fit` 스타일을 동적으로 제어하고, `classMap`과 `styleMap`으로 variant별 스타일을 적용한다.

## 의존성

### 외부 패키지
- `lit` — `html`, `nothing`, `css`
- `lit/decorators.js` — `customElement`
- `lit/directives/class-map.js` — `classMap`
- `lit/directives/style-map.js` — `styleMap`
- `@a2ui/web_core/v0_9/basic_catalog` — `ImageApi`
- `@a2ui/lit/v0_9` — `A2uiController`

### 저장소 내부 모듈
- [`../basic-catalog-a2ui-lit-element.ts`](../basic-catalog-a2ui-lit-element.ts.md) — `BasicCatalogA2uiLitElement`

## Exports

- `A2uiImageElement` — 클래스 (커스텀 엘리먼트)
- `A2uiImage` — 상수 (객체: `ImageApi` 스프레드 + `tagName: 'a2ui-image'`)

## 상세 명세

### `A2uiImageElement` 클래스

**데코레이터**: `@customElement('a2ui-image')`

**상속**: `BasicCatalogA2uiLitElement<typeof ImageApi>`

#### 정적 필드

- `static styles`: `CSSResult`:
  - `img`: `display: block`, `width: 100%`, `height: auto`, `border-radius: var(--a2ui-image-border-radius, 0)`.
  - `:host(.icon), img.icon`: 너비·높이 모두 `--a2ui-image-icon-size`(기본 `24px`).
  - `img.avatar`: 너비·높이 `--a2ui-image-avatar-size`(기본 `40px`), `border-radius: 50%`.
  - `:host(.smallFeature), img.smallFeature`: `max-width: --a2ui-image-small-feature-size`(기본 `100px`).
  - `:host(.largeFeature), img.largeFeature`: `max-height: --a2ui-image-large-feature-size`(기본 `400px`).
  - `:host(.header), img.header`: `height: --a2ui-image-header-size`(기본 `200px`), `object-fit: cover`.

#### `createController(): A2uiController`

`new A2uiController(this, ImageApi)`를 반환.

#### `render(): TemplateResult | typeof nothing`

1. `this.controller.props`가 없으면 `nothing` 반환.
2. `classes` 객체:
   - `'a2ui-image': true` 항상 포함.
   - `[props.variant || '']`: `props.variant`가 존재하면 해당 문자열 키를 `true`로 설정. 빈 문자열은 무시된다.
3. `styles` 객체: `{ objectFit: props.fit || 'fill' }`. `props.fit`이 없으면 `'fill'`을 기본값으로 사용한다.
4. `<img src={props.url} alt={props.description || ''} class={classMap(classes)} style={styleMap(styles)} />`를 반환한다.

### `A2uiImage` 상수

`ImageApi` 스프레드 + `tagName: 'a2ui-image'`.

## 동작 흐름

props에서 `url`, `variant`, `fit`, `description`을 추출 → variant를 클래스맵으로 변환하여 CSS 클래스 기반 크기 제어 → `fit`을 인라인 `object-fit` 스타일로 적용 → `<img>` 엘리먼트 렌더링.
