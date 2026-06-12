# renderers/react/visual-parity/fixtures/components/slider.ts

## 개요

Slider 컴포넌트의 비주얼 패리티 테스트용 단일 픽스처를 제공한다. A2UI 데이터 모델 패턴에 맞춰 `value`를 path 바인딩으로 연결하며, 초기값 50, 범위 0~100의 표준 슬라이더를 정의한다. 주석에 따르면 `label`은 A2UI Slider 명세의 표준 속성이 아니므로 사용하지 않는다.

## 의존성

### 저장소 내부 모듈

- [`../types`](../types.ts.md) — `ComponentFixture` 타입을 type-only import

### 외부 패키지

없음

## Exports

| 이름 | 종류 | 설명 |
|---|---|---|
| `slider` | 상수 (`ComponentFixture`) | 범위 0~100, 초기값 50인 Slider 픽스처 |
| `sliderFixtures` | 상수 (객체) | `slider`를 감싸는 그룹 객체 |

## 상세 명세

### `slider: ComponentFixture`

- `root`: `'slider-1'`
- `data`: `{'/slider/value': 50}` — 렌더 전에 데이터 모델의 `/slider/value` 경로에 50을 설정
- 단일 컴포넌트: `id: 'slider-1'`
  - `Slider.value`: `{path: '/slider/value'}` — 데이터 모델 경로에 바인딩
  - `Slider.minValue`: `0`
  - `Slider.maxValue`: `100`

### `sliderFixtures: object`

`slider` 픽스처를 단일 키-값으로 묶은 집계 객체.

## 동작 흐름

`data` 필드에 초기값이 선언되므로, 테스트 하네스는 렌더링 전에 `/slider/value: 50`을 데이터 모델에 주입해야 한다. Slider 컴포넌트는 해당 경로를 구독하여 슬라이더 위치를 50%로 표시한다. 픽스처는 단일 컴포넌트만 포함하므로 구성이 매우 단순하다.
