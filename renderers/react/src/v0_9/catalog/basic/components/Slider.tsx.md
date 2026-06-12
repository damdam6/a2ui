# renderers/react/src/v0_9/catalog/basic/components/Slider.tsx

## 개요

`Slider` 컴포넌트는 a2ui basic catalog의 `SliderApi` 스키마를 따르는 React 구현체다. HTML `<input type="range">`를 기반으로 슬라이더 UI를 제공하며, 레이블·현재 값 표시·스타일링을 모두 포함한다. 값 변경 시 `props.setValue` 콜백을 호출하는 제어 컴포넌트 패턴을 따른다.

## 의존성

### 외부 패키지
- `react` — `React`, `React.useId`, `React.CSSProperties`, `React.ChangeEvent`
- `@a2ui/web_core/v0_9/basic_catalog` — `SliderApi`

### 저장소 내부 모듈
- [`../../../adapter`](../../../adapter.tsx.md) — `createComponentImplementation`
- [`../utils`](../utils.ts.md) — `useBasicCatalogStyles`

## Exports

| 이름 | 종류 |
|------|------|
| `Slider` | 상수 (ReactComponentImplementation) |

## 상세 명세

### `Slider`

`createComponentImplementation(SliderApi, renderFn)` 호출로 생성된 컴포넌트 구현체다.

**렌더 함수 시그니처:** `({props}) => JSX.Element`

**매개변수 (props 필드):**
- `props.label` — 슬라이더 위에 표시할 레이블 문자열 (옵셔널)
- `props.value` — 현재 슬라이더 값 (숫자, 옵셔널). 없으면 `0` 사용
- `props.min` — 최솟값 (옵셔널). 없으면 `0` 사용
- `props.max` — 최댓값
- `props.setValue` — 값 변경 시 호출되는 콜백 `(value: number) => void`

**내부 로직:**

1. `useBasicCatalogStyles()`로 global 스타일을 DOM에 주입한다.
2. `onChange` 핸들러: `React.ChangeEvent<HTMLInputElement>` 이벤트를 받아 `Number(e.target.value)`를 `props.setValue`에 전달한다.
3. `React.useId()`로 `uniqueId`를 생성하여 `<label>`의 `htmlFor`와 `<input>`의 `id`를 연결한다.

**스타일 정의 (모두 `React.CSSProperties` 객체):**
- `containerStyle`: `flexDirection: 'column'`, gap은 `--a2ui-spacing-xs`(기본 `0.25rem`), margin은 `--a2ui-slider-margin` → `--a2ui-spacing-m`
- `headerStyle`: `display: 'flex'`, `justifyContent: 'space-between'`, `alignItems: 'center'`
- `labelStyle`: `fontSize`는 `--a2ui-slider-label-font-size` → `--a2ui-label-font-size` → `--a2ui-font-size-s`; `fontWeight`는 `--a2ui-slider-label-font-weight` → `--a2ui-label-font-weight` → `bold`
- `valueStyle`: `fontSize`는 `--a2ui-font-size-xs`(기본 `0.75rem`), `color`는 `--a2ui-text-caption-color`(기본 `light-dark(#666, #aaa)`)
- `inputStyle`: `width: '100%'`, `cursor: 'pointer'`, `accentColor`는 `--a2ui-slider-thumb-color` → `--a2ui-color-primary`(기본 `#007bff`), `background`는 `--a2ui-slider-track-color` → `--a2ui-color-secondary`(기본 `#e9ecef`)

**렌더 구조:**
- 최상위: `containerStyle`이 적용된 `<div>`
- 헤더 행: `headerStyle`이 적용된 `<div>` 안에:
  - `props.label`이 있을 때만 `<label htmlFor={uniqueId}>`를 `labelStyle`로 렌더링
  - 항상 `<span style={valueStyle}>{props.value}</span>` 렌더링
- `<input id={uniqueId} type="range" min={props.min ?? 0} max={props.max} value={props.value ?? 0} onChange={onChange} style={inputStyle} />`

## 동작 흐름

컴포넌트가 마운트되면 `uniqueId`가 생성되고, 슬라이더 드래그 → `onChange` → `props.setValue(Number(...))` 호출 → 상위 컴포넌트가 `props.value`를 업데이트 → 재렌더링으로 슬라이더 위치와 값 표시가 갱신된다.
