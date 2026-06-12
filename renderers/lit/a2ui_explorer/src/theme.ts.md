# renderers/lit/a2ui_explorer/src/theme.ts

## 개요

`a2ui_explorer`에서 사용할 v0.8 스타일 기반의 커스텀 테마를 정의하고 export하는 모듈이다. HTML 기본 요소(`a`, `button`, `input` 등)에 적용할 유틸리티 클래스 맵과, A2UI v0.8 컴포넌트(`Button`, `Card`, `TextField` 등)에 적용할 클래스 맵, 그리고 마크다운 렌더링에 사용할 클래스 배열을 포함한다. gradient 기반의 시각적으로 풍부한 디자인을 표현하기 위해 `additionalStyles`에 직접 CSS 속성을 지정한다.

## 의존성

### 외부 패키지
- `@a2ui/lit` — `v0_8` 네임스페이스 (`v0_8.Styles.merge`, `v0_8.Types.Theme` 타입)

### 저장소 내부 모듈
없음.

## Exports

- `theme` (`v0_8.Types.Theme`) — 전체 테마 객체

## 상세 명세

### HTML 요소 기본 스타일 상수 (모두 `Record<string, boolean>` 형태)

각 상수는 유틸리티 클래스명을 키로, `true`를 값으로 갖는 객체다. `v0_8.Styles.merge`에 전달하여 light 모드용 변형 객체(`*Light`)를 생성한다. 두 번째 인자로 빈 객체 `{}`를 전달하므로 오버라이드 없이 원본과 동일한 결과를 반환한다.

| 상수 | 주요 클래스 특성 |
|------|----------------|
| `a` | sans-serif 폰트(`typography-f-sf`), font-weight 500(`typography-w-500`), inline-flex(`layout-dis-iflx`), align-center(`layout-al-c`), text-decoration-none(`typography-td-none`), primary-40 색상(`color-c-p40`) |
| `audio` | 너비 100%(`layout-w-100`) |
| `body` | serif 폰트(`typography-f-s`), font-weight 400, margin-top 0, margin-bottom 2, body-medium 크기(`typography-sz-bm`), neutral-10 색상(`color-c-n10`) |
| `button` | sans-serif, font-weight 500, padding top/bottom 3, padding left/right 5, margin-bottom 1, border-radius 16, border-width 0, neutral-70 border color, solid border, secondary-30 배경(`color-bgc-s30`), hover opacity 80 |
| `heading` | sans-serif, font-weight 500, margin-top 0, margin-bottom 2 |
| `iframe` | scroll-behavior none(`behavior-sw-n`) |
| `input` | sans-serif, font-weight 400, padding 4/2, border-radius 6, border-width 1, secondary-70 border color(`color-bc-s70`), solid border, align-self normal, neutral-10 텍스트 |
| `p` | serif, font-weight 400, margin 0, body-medium 크기, align-self normal, neutral-10 색상 |
| `orderedList` | `p`와 동일 |
| `unorderedList` | `p`와 동일 |
| `listItem` | `p`와 동일 |
| `pre` | monospace 폰트(`typography-f-c`), font-weight 400, body-medium 크기, white-space pre(`typography-ws-p`), align-self normal |
| `textarea` | `input` 스프레드 + resize none(`layout-r-none`) + flex-shrink contain(`layout-fs-c`) |
| `video` | object-fit cover(`layout-el-cv`) |

`*Light` 변형 상수: `aLight`, `inputLight`, `textareaLight`, `buttonLight`, `bodyLight`, `pLight`, `preLight`, `orderedListLight`, `unorderedListLight`, `listItemLight` — 각각 `v0_8.Styles.merge(원본, {})` 호출 결과.

### `theme` 상수 (`v0_8.Types.Theme`)

세 가지 최상위 키를 가진다.

#### `additionalStyles`

컴포넌트 이름을 키로, 직접 CSS 속성 객체를 값으로 가지는 맵. 주요 내용:

- **`Button`**: CSS 변수 `--n-35`를 `var(--n-100)`으로, `--n-10`을 `var(--n-0)`으로 재정의. `linear-gradient(135deg, light-dark(#818cf8, #06b6d4) 0%, light-dark(#a78bfa, #3b82f6) 100%)` 배경. `boxShadow: '0 4px 15px rgba(102, 126, 234, 0.4)'`. `padding: '12px 28px'`. `textTransform: 'uppercase'`.
- **`Text`** (h1~h3): `color: 'transparent'`에 동일한 135도 그라디언트 배경을 `-webkit-background-clip: text`, `background-clip: text`, `-webkit-text-fill-color: transparent`로 적용하여 그라디언트 텍스트 효과. h4, h5, body, caption은 빈 객체.
- **`Card`**: `radial-gradient(circle at top left, ...)` + `radial-gradient(circle at bottom right, ...)` + `linear-gradient(135deg, ...)` 조합의 글래스모피즘 배경.
- **`TextField`**: CSS 변수 `--p-0`을 `light-dark(var(--n-0), #1e293b)`로 재정의.
- **`Modal`**: `linear-gradient(135deg, ...)` 반투명 배경. `boxShadow: '0 10px 25px -5px rgba(0,0,0,0.3), ...'`. `borderRadius: '8px'`. `padding: '16px'`. `minWidth: '300px'`. `maxWidth: '80vw'`. `display: 'flex'`. `flexDirection: 'column'`.
- **`MultipleChoice`**: `--md-sys-color-secondary-container-high: '#e8def8'`.

#### `components`

A2UI 컴포넌트 이름을 키로, 유틸리티 클래스 맵(또는 sub-key 맵)을 값으로 가지는 맵. 주요 내용:

- **`Button`**: `layout-pt-2`, `layout-pb-2`, `layout-pl-3`, `layout-pr-3`, `border-br-12`, `border-bw-0`, `border-bs-s`, `color-bgc-p30`, `behavior-ho-70`, `typography-w-400`.
- **`Card`**: `border-br-9`, `layout-p-4`, `color-bgc-n100`.
- **`CheckBox`**: `element`(체크박스 엘리먼트 — margin, padding, border-radius 12, border-width 1, solid border, primary-100 배경, primary-60 border, neutral-30/primary-30 텍스트), `label`(primary-30 색상, sf폰트, font-weight 400, flex 1, large-large 크기), `container`(inline-flex, align-center) 세 sub-key로 구성.
- **`Column`**: `layout-g-2`.
- **`DateTimeInput`**: `container`(body-medium 크기, width 100%, gap 2, flex row, align center, white-space nowrap), `label`(primary-30 색상, body-medium 크기), `element`(padding 2/3, border-radius 2, border-width 1, solid border, primary-100 배경, primary-60 border, neutral-30/primary-30 텍스트) sub-key.
- **`Divider`**: 빈 객체.
- **`Image`**: `all`(border-radius 5, object-fit cover, width/height 100%), `avatar`(`is-avatar`), `header`/`icon`/`largeFeature`/`mediumFeature`/`smallFeature`는 빈 객체.
- **`Icon`**: 빈 객체.
- **`List`**: `layout-g-4`, `layout-p-2`.
- **`Modal`**: `backdrop`(`color-bbgc-p60_20`), `element`(`border-br-2`, `color-bgc-p100`, `layout-p-4`, `border-bw-1`, `border-bs-s`, `color-bc-p80`) sub-key.
- **`MultipleChoice`**: `container`, `label`, `element` 모두 빈 객체.
- **`Row`**: `layout-g-4`.
- **`Slider`**: `container`, `label`, `element` 모두 빈 객체.
- **`Tabs`**: `container`, `controls`(all/selected 빈 객체), `element` 모두 빈 객체.
- **`Text`**: `all`(`layout-w-100`, `layout-g-2`), `h1`(sf폰트, font-weight 400, margin/padding 0, `typography-sz-hs`), `h2`(sf폰트, font-weight 400, margin/padding 0, `typography-sz-tl`), `h3`(`typography-sz-tl`), `h4`(`typography-sz-bl`), `h5`(`typography-sz-bm`), `body`와 `caption`은 빈 객체.
- **`TextField`**: `container`(body-medium 크기, width 100%, gap 2, flex row, align center, white-space nowrap), `label`(flex 0, primary-30 색상), `element`(body-medium, padding 2/3, border-radius 2, border-width 1, solid border, primary-100 배경, primary-60 border, neutral-30/primary-30 텍스트) sub-key.
- **`Video`**: `border-br-5`, `layout-el-cv`.

#### `elements`

HTML 요소 태그명을 키로, `*Light` 변형 상수를 값으로 가지는 맵:

`a: aLight`, `audio: audio`, `body: bodyLight`, `button: buttonLight`, `h1~h5: heading`, `iframe: iframe`, `input: inputLight`, `p: pLight`, `pre: preLight`, `textarea: textareaLight`, `video: video`.

#### `markdown`

마크다운 렌더러가 사용할 태그별 클래스 이름 배열. `Object.keys()`로 `*Light` 상수에서 키를 추출하여 배열로 변환한다:

`p: Object.keys(pLight)`, `h1~h5: Object.keys(heading)`, `ul: Object.keys(unorderedListLight)`, `ol: Object.keys(orderedListLight)`, `li: Object.keys(listItemLight)`, `a: Object.keys(aLight)`. `strong`과 `em`은 빈 배열 `[]`.

## 동작 흐름

파일 최상단에서 HTML 요소별 기본 클래스 맵 상수들이 정의된다. 이어서 `v0_8.Styles.merge`를 통해 light 모드용 변형들이 계산된다. 마지막으로 이들을 조합하여 `theme` 객체가 구성된다. 이 파일 자체에는 런타임 실행 흐름이 없으며, import 시 평가된 `theme` 상수를 소비자(예: `LocalGallery` 또는 다른 Lit 컴포넌트)가 A2UI 렌더러에 주입하여 사용한다.
