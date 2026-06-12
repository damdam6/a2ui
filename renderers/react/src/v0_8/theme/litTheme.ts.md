# renderers/react/src/v0_8/theme/litTheme.ts

## 개요

A2UI React 컴포넌트의 기본 테마를 정의하는 파일이다. Lit 렌더러의 내부 스타일링과 동일한 CSS 유틸리티 클래스 규칙을 사용하여 React와 Lit 구현 간 시각적 일관성을 보장한다. `Types.Theme` 타입을 구현하는 `litTheme` 객체를 내보내며, 이는 `defaultTheme`이라는 별칭으로도 내보낸다.

## 의존성

### 외부 패키지

- `@a2ui/web_core/types/types` (타입 전용) — `Types.Theme` 타입

### 저장소 내부 모듈

없음.

## Exports

| 이름 | 종류 | 설명 |
|------|------|------|
| `litTheme` | 상수 (`Types.Theme`) | A2UI React 기본 테마 객체 |
| `defaultTheme` | 상수 (`Types.Theme`) | `litTheme`의 별칭 |

## 상세 명세

### 파일 내부 Element 스타일 상수들

HTML 요소별 클래스맵(`Record<string, boolean>`)을 미리 정의한 모듈-레벨 상수들. 이후 `litTheme.elements`와 `litTheme.markdown`에서 재사용된다.

| 상수명 | 주요 클래스 |
|--------|------------|
| `elementA` | `typography-f-sf`, `typography-w-500`, `layout-dis-iflx`, `typography-td-none`, `color-c-p40` |
| `elementAudio` | `layout-w-100` |
| `elementBody` | `typography-f-s`, `typography-w-400`, `layout-mt-0`, `layout-mb-2`, `typography-sz-bm`, `color-c-n10` |
| `elementButton` | `typography-f-sf`, `typography-w-500`, `layout-pt-3` ~ `layout-pr-5`, `border-br-16`, `color-bgc-s30`, `behavior-ho-80` |
| `elementHeading` | `typography-f-sf`, `typography-w-500`, `layout-mt-0`, `layout-mb-2` |
| `elementIframe` | `behavior-sw-n` |
| `elementInput` | `typography-f-sf`, `layout-pl-4` ~ `layout-pb-2`, `border-br-6`, `border-bw-1`, `color-bc-s70`, `color-c-n10` |
| `elementP` | `typography-f-s`, `typography-w-400`, `layout-m-0`, `typography-sz-bm`, `color-c-n10` |
| `elementList` | `elementP`와 동일한 클래스 세트 |
| `elementPre` | `typography-f-c`, `typography-w-400`, `typography-sz-bm`, `typography-ws-p` |
| `elementTextarea` | `elementInput` 전체를 스프레드 후 `layout-r-none`, `layout-fs-c` 추가 |
| `elementVideo` | `layout-el-cv` |

모든 상수는 `boolean` 값이 `true`인 키만 클래스로 적용된다.

### `litTheme: Types.Theme`

`Types.Theme` 타입을 구현하는 객체로 세 최상위 섹션을 가진다.

#### `litTheme.components`

각 컴포넌트에 적용할 클래스맵을 정의한다. 컴포넌트 이름이 키이며 값은 클래스맵 또는 하위 변형(variant) 맵이다.

**Content 컴포넌트:**
- `AudioPlayer`: `{}` (빈 스타일)
- `Divider`: `{}` (빈 스타일)
- `Icon`: `{}` (빈 스타일)
- `Image`: `all`에 `border-br-5`, `layout-el-cv`, `layout-w-100`, `layout-h-100`; `avatar`에 `is-avatar: true`; 나머지 변형(`header`, `icon`, `largeFeature`, `mediumFeature`, `smallFeature`)은 빈 맵
- `Text`: `all`에 `layout-w-100`, `layout-g-2`; 헤딩(`h1`~`h5`)별로 `typography-f-sf`, `typography-v-r`, `typography-w-400`, 마진/패딩 제거, 크기 클래스(`h1`: `typography-sz-hs`, `h2/h3`: `typography-sz-tl`, `h4`: `typography-sz-bl`, `h5`: `typography-sz-bm`); `body`, `caption`은 빈 맵
- `Video`: `border-br-5`, `layout-el-cv`

**Layout 컴포넌트:**
- `Card`: `border-br-9`, `layout-p-4`, `color-bgc-n100`
- `Column`: `layout-g-2`
- `List`: `layout-g-4`, `layout-p-2`
- `Modal`: `backdrop`에 `color-bbgc-p60_20`; `element`에 `border-br-2`, `color-bgc-p100`, `layout-p-4`, `border-bw-1`, `border-bs-s`, `color-bc-p80`
- `Row`: `layout-g-4`
- `Tabs`: `container`, `controls.all`, `controls.selected`, `element` 모두 빈 맵

**Interactive 컴포넌트:**
- `Button`: `layout-pt-2` ~ `layout-pr-3`, `border-br-12`, `border-bw-0`, `color-bgc-p30`, `color-c-p100`(흰색 텍스트), `behavior-ho-70`, `typography-w-400`
- `CheckBox`: `container`에 `layout-dsp-iflex`, `layout-al-c`; `element`에 `border-br-12`, `border-bw-1`, `color-bgc-p100`, `color-bc-p60`, `color-c-n30`, `color-c-p30`; `label`에 `color-c-p30`, `typography-f-sf`, `typography-v-r`, `layout-flx-1`, `typography-sz-ll`
- `DateTimeInput`: `container`에 `typography-sz-bm`, `layout-dsp-flexhor`, `layout-al-c`, `typography-ws-nw`; `label`에 `color-c-p30`, `typography-sz-bm`; `element`에 `layout-pt-2` ~ `layout-pr-3`, `border-br-2`, `color-bgc-p100`, `color-bc-p60`
- `MultipleChoice`, `Slider`: `container`, `label`, `element` 모두 빈 맵
- `TextField`: `container`에 `typography-sz-bm`, `layout-dsp-flexhor`, `layout-al-c`, `typography-ws-nw`; `label`에 `layout-flx-0`, `color-c-p30`; `element`에 `typography-sz-bm`, `border-br-2`, `border-bw-1`, `color-bgc-p100`, `color-bc-p60`, `color-c-n30`, `color-c-p30`

#### `litTheme.elements`

HTML 요소 태그명을 키로 하는 맵. 적용 대상: `a`, `audio`, `body`, `button`, `h1`~`h5`(모두 `elementHeading`), `iframe`, `input`, `p`, `pre`, `textarea`, `video`. 각각 위에서 정의한 element 상수들을 값으로 사용한다.

#### `litTheme.markdown`

markdown-it 렌더러에 전달할 클래스 배열 맵. `Object.keys(elementX)` 패턴으로 클래스 이름 배열을 추출한다. 적용 대상: `p`, `h1`~`h5`(모두 `elementHeading` 키 배열), `ul`, `ol`, `li`(모두 `elementList` 키 배열), `a`(`elementA` 키 배열), `strong`(`[]`), `em`(`[]`).

### `defaultTheme: Types.Theme`

`litTheme`을 그대로 재내보내는 별칭 상수. `ThemeContext.tsx`에서 기본값 폴백으로 import한다.

## 동작 흐름

이 파일은 모듈 로드 시 모든 상수를 즉시 계산한다. `litTheme`의 `markdown` 섹션은 `elements` 섹션과 동일한 element 상수를 참조하여 `Object.keys()`로 클래스 배열을 생성한다. 결과 객체는 불변 상수로 `ThemeContext`의 기본 테마로 사용되며, 컴포넌트들이 `useTheme()` 훅을 통해 소비한다.
