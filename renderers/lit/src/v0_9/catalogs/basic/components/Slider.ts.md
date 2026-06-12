# renderers/lit/src/v0_9/catalogs/basic/components/Slider.ts

## 개요

`a2ui-slider` 커스텀 엘리먼트를 정의하는 파일이다. HTML5 `<input type="range">`를 래핑하여 레이블, 현재 값 표시, 최솟값·최댓값 설정을 지원하는 슬라이더 UI를 렌더링한다. 사용자 드래그 입력을 `props.setValue`로 즉시 전파한다.

## 의존성

### 외부 패키지
- `lit` — `html`, `nothing`, `css`
- `lit/decorators.js` — `customElement`
- `@a2ui/web_core/v0_9/basic_catalog` — `SliderApi`
- `@a2ui/lit/v0_9` — `A2uiController`

### 저장소 내부 모듈
- [`../basic-catalog-a2ui-lit-element.ts`](../basic-catalog-a2ui-lit-element.ts.md) — `BasicCatalogA2uiLitElement`

## Exports

- `A2uiSliderElement` — 클래스 (커스텀 엘리먼트)
- `A2uiSlider` — 상수 (객체: `SliderApi` 스프레드 + `tagName: 'a2ui-slider'`)

## 상세 명세

### `A2uiSliderElement` 클래스

**데코레이터**: `@customElement('a2ui-slider')`

**상속**: `BasicCatalogA2uiLitElement<typeof SliderApi>`

#### 정적 필드

- `static styles`: `CSSResult`:
  - `:host`: `display: flex`, `flex-direction: column`, `gap: --a2ui-spacing-xs`, `margin: --a2ui-slider-margin`(기본 `--a2ui-spacing-m`).
  - `.header`: `display: flex`, `justify-content: space-between`, `align-items: center` — 레이블과 현재 값을 양 끝에 배치한다.
  - `.header label`: `font-size: --a2ui-slider-label-font-size`(폴백 `--a2ui-label-font-size` → `--a2ui-font-size-s`), `font-weight: --a2ui-slider-label-font-weight`(폴백 `--a2ui-label-font-weight` → `bold`).
  - `input[type='range']`: `width: 100%`, `accent-color: --a2ui-slider-thumb-color`(기본 `--a2ui-color-primary` → `#007bff`), `background: --a2ui-slider-track-color`(기본 `--a2ui-color-secondary` → `#e9ecef`).

#### `createController(): A2uiController`

`new A2uiController(this, SliderApi)`를 반환.

#### `render(): TemplateResult | typeof nothing`

1. `this.controller.props`가 없으면 `nothing` 반환.
2. 헤더 영역 `<div class="header">` 렌더링:
   - `props.label`이 있으면 `<label>{props.label}</label>` 출력.
   - `<span>{props.value}</span>`으로 현재 값 표시.
3. `<input type="range">` 렌더링:
   - `min`: `props.min ?? 0`
   - `max`: `props.max ?? 100`
   - `.value`: `props.value?.toString() || '0'` — Lit의 `.value` 바인딩으로 DOM 프로퍼티에 직접 설정.
   - `@input`: `props.setValue?.(Number(e.target.value))` — 이벤트 발생 시 `Number`로 변환하여 전달.

### `A2uiSlider` 상수

`SliderApi` 스프레드 + `tagName: 'a2ui-slider'`.

## 동작 흐름

props 수신 → 헤더(레이블 + 현재 값)와 range input 렌더링 → 사용자가 슬라이더를 드래그하면 `@input` 이벤트로 `props.setValue`에 숫자 값 전달. `props.value`는 `toString()`으로 변환하여 `<input>`의 DOM 프로퍼티에 직접 설정하므로 외부 상태 변경 시 올바르게 반영된다.
