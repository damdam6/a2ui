# renderers/react/visual-parity/fixtures/themes/minimalTheme.ts

## 개요

시각적 동등성 테스트에서 스타일 간섭을 최소화하고 구조적 정확성만을 검증하기 위해 설계된 최소(minimal) 테마를 정의한다. 중립적인 색상과 기초적인 스타일만 사용하며, `Types.Theme` 인터페이스를 준수하여 `components`, `elements`, `markdown` 세 섹션을 모두 포함한다.

## 의존성

### 외부 패키지

- `@a2ui/lit/0.8` — `Types` (네임스페이스, `Types.Theme` 타입 사용)

### 저장소 내부 모듈

없음.

## Exports

| 이름 | 종류 | 설명 |
|---|---|---|
| `minimalTheme` | 상수 (`Types.Theme`) | 최소 스타일 테마 객체 |

## 상세 명세

### `minimalTheme: Types.Theme`

`Types.Theme` 타입의 객체 상수. 세 최상위 섹션을 가진다.

#### `components` 섹션

각 컴포넌트 이름을 키로 하며, 값은 CSS 유틸리티 클래스명을 키로 `true`를 값으로 갖는 객체이다(Lit A2UI의 클래스 매핑 방식).

| 컴포넌트 | 적용 클래스 | 비고 |
|---|---|---|
| `Text.all` | `layout-w-100` | 전체 Text에 전체 너비 적용 |
| `Text.h1` | `typography-w-600`, `typography-sz-hl`, `layout-m-0` | |
| `Text.h2` | `typography-w-600`, `typography-sz-hm`, `layout-m-0` | |
| `Text.h3` | `typography-w-600`, `typography-sz-hs`, `layout-m-0` | |
| `Text.h4` | `typography-w-500`, `typography-sz-tl`, `layout-m-0` | |
| `Text.h5` | `typography-w-500`, `typography-sz-tm`, `layout-m-0` | |
| `Text.body` | (빈 객체) | |
| `Text.caption` | `typography-sz-bs`, `color-c-n50` | |
| `Icon` | (빈 객체) | |
| `Image.all` | (빈 객체) | |
| `Image.avatar` | `border-br-50` | 원형 아바타 |
| `Image.header`, `icon`, `largeFeature`, `mediumFeature`, `smallFeature` | (빈 객체) | |
| `Divider` | (빈 객체) | |
| `AudioPlayer` | (빈 객체) | |
| `Video` | (빈 객체) | |
| `Card` | `layout-p-3`, `border-br-4`, `border-bw-1`, `color-bc-n80` | |
| `Row` | `layout-g-2` | |
| `Column` | `layout-g-2` | |
| `List` | `layout-g-2` | |
| `Modal.backdrop` | (빈 객체) | |
| `Modal.element` | `layout-p-3`, `border-br-4` | |
| `Tabs.element` | (빈 객체) | |
| `Tabs.controls.all` | (빈 객체) | |
| `Tabs.controls.selected` | (빈 객체) | |
| `Tabs.container` | (빈 객체) | |
| `Button` | `layout-pt-2`, `layout-pb-2`, `layout-pl-3`, `layout-pr-3`, `border-br-4`, `border-bw-1`, `color-bc-n70` | |
| `CheckBox.container` | `layout-al-c`, `layout-g-2` | |
| `CheckBox.element`, `label` | (빈 객체) | |
| `TextField.container` | `layout-g-1` | |
| `TextField.label` | (빈 객체) | |
| `TextField.element` | `layout-p-2`, `border-br-4`, `border-bw-1`, `color-bc-n70` | |
| `Slider.container` | `layout-g-1` | |
| `Slider.label`, `element` | (빈 객체) | |
| `DateTimeInput.container` | `layout-g-1` | |
| `DateTimeInput.label` | (빈 객체) | |
| `DateTimeInput.element` | `layout-p-2`, `border-br-4`, `border-bw-1`, `color-bc-n70` | |
| `MultipleChoice.container` | `layout-g-2` | |
| `MultipleChoice.label`, `element` | (빈 객체) | |

#### `elements` 섹션

HTML 네이티브 엘리먼트(`a`, `audio`, `body`, `button`, `h1`~`h5`, `iframe`, `input`, `p`, `pre`, `textarea`, `video`) 모두 빈 객체로 정의된다. 네이티브 요소에 추가 클래스를 적용하지 않는다.

#### `markdown` 섹션

마크다운 렌더링 시 적용할 클래스 배열 매핑이다.

- `p`, `h1`~`h5`: `['layout-m-0']` — 마진을 제거하여 비마크다운 렌더링과 동일하게 정렬
- `ul`, `ol`, `li`, `a`, `strong`, `em`: `[]` — 추가 클래스 없음

## 동작 흐름

모듈 로드 시 `minimalTheme` 상수가 단 한 번 생성된다. 이후 `themes/index.ts`에서 레지스트리에 등록되어, 테스트 하네스가 `getTheme('minimal')`을 호출할 때 반환된다.
