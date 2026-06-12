# renderers/react/visual-parity/fixtures/themes/visualParityTheme.ts

## 개요

테마 전환(theme switching) 동작을 검증하기 위해 설계된 대체(alternate) 테마를 정의한다. `minimalTheme`보다 풍부한 스타일을 가지며, serif 폰트 계열·고정 색상·flexbox 레이아웃 유틸리티 등을 적극 활용한다. `Types.Theme` 인터페이스를 준수하여 `components`, `elements`, `markdown` 세 섹션을 완전히 정의한다.

## 의존성

### 외부 패키지

- `@a2ui/lit/0.8` — `Types` (네임스페이스, `Types.Theme` 타입 사용)

### 저장소 내부 모듈

없음.

## Exports

| 이름 | 종류 | 설명 |
|---|---|---|
| `visualParityTheme` | 상수 (`Types.Theme`) | 시각적 동등성 검증용 대체 테마 객체 |

## 상세 명세

### `visualParityTheme: Types.Theme`

`Types.Theme` 타입의 객체 상수. 세 최상위 섹션을 가진다.

#### `components` 섹션

##### `Text`

- `all`: `typography-f-s`(serif), `typography-fs-n`, `typography-w-400`, `typography-sz-bm`, `color-c-n10`, `layout-w-100`
- `h1`~`h3`: `typography-f-sf`(sans-serif), `typography-w-500`, 크기 각각 `sz-hl` / `sz-hm` / `sz-hs`
- `h4`, `h5`: `typography-f-sf`, `typography-w-500`, 크기 각각 `sz-tl` / `sz-tm`
- `body`: 빈 객체
- `caption`: `typography-sz-ls`, `color-c-n40`

##### `Icon`

`g-icon: true`, `filled-heavy: true` — Material Icons 스타일 적용.

##### `Image`

- `all`: `layout-w-100`
- `avatar`: `border-br-50` (원형)
- 그 외 용도(`header`, `icon`, `largeFeature`, `mediumFeature`, `smallFeature`): 빈 객체

##### `Button`

`layout-dis-iflx`, `layout-al-c`, `layout-jc-c`, `layout-g-2`, `layout-pt-3`, `layout-pb-3`, `layout-pl-5`, `layout-pr-5`, `border-br-16`, `border-bw-0`, `color-bgc-p40`, `color-c-n100`, `typography-f-sf`, `typography-w-500`

##### `Card`

`layout-p-4`, `layout-dis-flx`, `layout-fd-c`, `layout-g-3`, `border-br-12`, `color-bgc-n98`

##### `Row`

`layout-dis-flx`, `layout-fd-r`, `layout-g-2`, `layout-w-100`

##### `Column`

`layout-dis-flx`, `layout-fd-c`, `layout-g-2`, `layout-w-100`

##### `List`

`layout-dis-flx`, `layout-fd-c`, `layout-g-2`, `layout-w-100`

##### `Divider`

`layout-w-100`, `color-bgc-n90`. 주석에 따르면 `layout-h-1` 클래스는 존재하지 않아 제거됨(`layout-h-10` 이상만 존재).

##### `TextField`

- `container`: `layout-dis-flx`, `layout-fd-c`, `layout-g-1`
- `label`: `typography-f-sf`, `typography-sz-bm`, `color-c-n40`
- `element`: `layout-p-3`, `border-br-8`, `border-bw-1`, `color-bc-n80`

##### `CheckBox`

- `container`: `layout-dis-flx`, `layout-al-c`, `layout-g-2`
- `element`: 빈 객체
- `label`: `typography-f-s`, `typography-sz-bm`

##### `Slider`

- `container`: `layout-dis-flx`, `layout-fd-c`, `layout-g-1`, `layout-w-100`
- `label`: `typography-f-sf`, `typography-sz-bm`, `color-c-n40`
- `element`: `layout-w-100`

##### `DateTimeInput`

- `container`: `layout-dis-flx`, `layout-fd-c`, `layout-g-1`
- `label`: `typography-f-sf`, `typography-sz-bm`, `color-c-n40`
- `element`: `layout-p-3`, `border-br-8`, `border-bw-1`, `color-bc-n80`

##### `MultipleChoice`

- `container`: `layout-dis-flx`, `layout-fd-c`, `layout-g-2`
- `label`: `typography-f-sf`, `typography-sz-bm`
- `element`: 빈 객체

##### `AudioPlayer`

`layout-w-100`

##### `Video`

`layout-w-100`

##### `Modal`

- `backdrop`: `layout-pos-f`, `layout-t-0`, `layout-l-0`, `layout-w-100vw`, `layout-h-100vh`, `color-bgc-n0`, `opacity-el-50`
- `element`: `layout-p-4`, `border-br-12`, `color-bgc-n100`

##### `Tabs`

- `element`: `layout-dis-flx`, `layout-fd-c`, `layout-g-2`
- `controls.all`: `layout-p-2`
- `controls.selected`: `color-bgc-p90`, `border-br-8`
- `container`: `layout-dis-flx`, `layout-g-2`

#### `elements` 섹션

`a`, `audio`, `body`, `button`, `h1`~`h5`, `iframe`, `input`, `p`, `pre`, `textarea`, `video` 모두 빈 객체.

#### `markdown` 섹션

- `p`, `h1`~`h5`: `['layout-m-0']` — 마진 제거
- `ul`, `ol`, `li`, `a`, `strong`, `em`: `[]`

## 동작 흐름

모듈 로드 시 `visualParityTheme` 상수가 생성되고, `themes/index.ts`의 레지스트리에 `'visualParity'` 키로 등록된다. 렌더러 진입점이 URL 파라미터 `?theme=visualParity`를 수신할 때 `getTheme('visualParity')`를 통해 이 객체가 반환된다.
