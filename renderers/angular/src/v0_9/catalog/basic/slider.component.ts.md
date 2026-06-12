# renderers/angular/src/v0_9/catalog/basic/slider.component.ts

## 개요

A2UI v0.9 Slider 컴포넌트의 Angular 구현체다. `<input type="range">`를 사용하여 범위 슬라이더를 렌더링하며, 레이블과 현재 값을 슬라이더 상단에 함께 표시한다. 사용자 조작 시 `onUpdate` 콜백을 통해 데이터 모델을 업데이트한다.

## 의존성

### 외부 패키지
- `@angular/core`: `Component`, `computed`, `ChangeDetectionStrategy`
- `@a2ui/web_core/v0_9/basic_catalog`: `SliderApi`

### 저장소 내부 모듈
- [`./basic-catalog-component`](./basic-catalog-component.ts.md): 기반 추상 클래스

## Exports

- `SliderComponent` — Angular standalone 컴포넌트 클래스

## 상세 명세

### `SliderComponent`

**데코레이터**: `@Component`
- `selector`: `'a2ui-v09-slider'`
- `standalone: true`
- `imports`: `[]`
- `changeDetection`: `ChangeDetectionStrategy.OnPush`

**상속**: `BasicCatalogComponent<typeof SliderApi>`

**computed 필드**

| 이름 | 타입 | 소스 | 기본값 |
|------|------|------|--------|
| `label` | `Signal<string \| undefined>` | `props()['label']?.value()` | — |
| `value` | `Signal<number \| undefined>` | `props()['value']?.value()` | — |
| `min` | `Signal<number>` | `props()['min']?.value() ?? 0` | `0` |
| `max` | `Signal<number>` | `props()['max']?.value() ?? 100` | `100` |
| `step` | `Signal<number>` | `props()['step']?.value() ?? 1` | `1` |

**메서드**

#### `handleInput(event: Event): void`
`input` DOM 이벤트를 처리한다. `(event.target as HTMLInputElement).value`를 `Number()`로 변환한 뒤 `this.props()['value']?.onUpdate(val)`을 호출하여 A2UI 데이터 모델을 업데이트한다. `onUpdate`가 없는 경우(읽기 전용 props)는 호출이 생략된다.

**템플릿 구조**

1. `div.a2ui-slider-container` 안에 `div.a2ui-slider-header`와 `input[type="range"]`를 세로로 배치한다.
2. `a2ui-slider-header`에는 `span.a2ui-slider-label`(레이블 텍스트)과 `span.a2ui-slider-value`(현재 값)가 가로로 배치된다.
3. `input[type="range"]`에 `[min]`, `[max]`, `[step]`, `[value]`를 바인딩하고 `(input)` 이벤트에 `handleInput($event)`를 연결한다.

**스타일 (CSS 변수 지원)**

| CSS 변수 | 기본값 | 역할 |
|----------|--------|------|
| `--a2ui-slider-margin` | `var(--a2ui-spacing-m, 16px)` | 컨테이너 마진 |
| `--a2ui-slider-label-font-size` | `var(--a2ui-label-font-size, var(--a2ui-font-size-s, 14px))` | 레이블 폰트 크기 |
| `--a2ui-slider-label-font-weight` | `bold` | 레이블 폰트 굵기 |
| `--a2ui-slider-thumb-color` | `var(--a2ui-color-primary, #007bff)` | 슬라이더 thumb 색상(`accent-color`) |
| `--a2ui-slider-track-color` | `var(--a2ui-color-secondary, #e9ecef)` | 슬라이더 트랙 배경색 |

헤더 내부 레이블과 값 텍스트는 `justify-content: space-between`으로 좌우에 배치된다.

## 동작 흐름

컴포넌트가 초기화되면 `props()`에서 `label`, `value`, `min`, `max`, `step`을 computed signal로 읽는다. `min`/`max`/`step`은 값이 없을 때 각각 `0`, `100`, `1`을 기본값으로 사용한다. 렌더링된 `input[type="range"]`의 현재 값이 변경되면 `handleInput`이 숫자로 변환하여 `onUpdate`를 통해 데이터 모델에 즉시 반영한다.
