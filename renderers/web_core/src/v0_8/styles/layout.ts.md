# renderers/web_core/src/v0_8/styles/layout.ts

## 개요

padding, margin, gap, position, display, flex, grid, width, height 등 레이아웃 전반에 관한 CSS 유틸리티 클래스들을 문자열로 생성하여 export한다. 픽셀 값은 `shared.ts`의 `grid` 상수(4px)를 기반으로 계산된다. 음수 값을 포함한 넓은 범위의 간격 클래스와 고정 픽셀 치수 클래스를 프로그래밍적으로 생성한다.

## 의존성

### 저장소 내부 모듈
- [`./shared.ts`](./shared.ts.md) — `grid` (값: `4`)

## Exports

| 이름 | 종류 |
|------|------|
| `layout` | 상수 (`string`) |

## 상세 명세

### `layout`

export되는 CSS 문자열 상수. 다음 섹션들로 구성된다:

**CSS 변수 선언 (`:host` 블록)**
`--g-1`~`--g-16`: `(idx + 1) * grid`px 값. `idx` = 0~15이므로 `--g-1: 4px` ~ `--g-16: 64px`.

**간격 클래스 (49개 × 10개 속성)**
`index` = 0~48, `idx = index - 24` (즉 -24 ~ 24). 음수 값은 `n` 접두어를 사용한다 (예: `idx = -3` → `lbl = 'n3'`). 양수 및 0은 숫자 그대로 (예: `idx = 0` → `lbl = '0'`).

각 `idx`에 대해 생성되는 클래스 (값: `idx * grid`px = `idx * 4`px):
- `.layout-p-${lbl}` — `--padding: ${idx*4}px; padding: var(--padding)` (padding 변수도 함께 설정)
- `.layout-pt-${lbl}`, `.layout-pr-${lbl}`, `.layout-pb-${lbl}`, `.layout-pl-${lbl}` — 방향별 padding
- `.layout-m-${lbl}` — `--margin: ${idx*4}px; margin: var(--margin)`
- `.layout-mt-${lbl}`, `.layout-mr-${lbl}`, `.layout-mb-${lbl}`, `.layout-ml-${lbl}` — 방향별 margin
- `.layout-t-${lbl}`, `.layout-r-${lbl}`, `.layout-b-${lbl}`, `.layout-l-${lbl}` — top/right/bottom/left 위치값

**gap 클래스 (25개)**
`idx` = 0~24: `.layout-g-${idx}` — `gap: ${idx * 4}px`

**grid-template-columns 클래스 (8개)**
`idx` = 0~7: `.layout-grd-col${idx+1}` — `grid-template-columns: ${'1fr '.repeat(idx+1).trim()}`
예: `.layout-grd-col3`은 `grid-template-columns: 1fr 1fr 1fr`

**position 클래스**
- `.layout-pos-a` — `position: absolute`
- `.layout-pos-rel` — `position: relative`

**display 클래스**
- `.layout-dsp-none` — `display: none`
- `.layout-dsp-block` — `display: block`
- `.layout-dsp-grid` — `display: grid`
- `.layout-dsp-iflex` — `display: inline-flex`
- `.layout-dsp-flexvert` — `display: flex; flex-direction: column`
- `.layout-dsp-flexhor` — `display: flex; flex-direction: row`

**flex 관련 클래스**
- `.layout-fw-w` — `flex-wrap: wrap`
- `.layout-al-fs` — `align-items: start`
- `.layout-al-fe` — `align-items: end`
- `.layout-al-c` — `align-items: center`
- `.layout-as-n` — `align-self: normal`
- `.layout-js-c` — `justify-self: center`
- `.layout-sp-c` — `justify-content: center`
- `.layout-sp-ev` — `justify-content: space-evenly`
- `.layout-sp-bt` — `justify-content: space-between`
- `.layout-sp-s` — `justify-content: start`
- `.layout-sp-e` — `justify-content: end`
- `.layout-ji-e` — `justify-items: end`
- `.layout-flx-0` — `flex: 0 0 auto`
- `.layout-flx-1` — `flex: 1 0 auto`

**기타 레이아웃 클래스**
- `.layout-r-none` — `resize: none`
- `.layout-fs-c` — `field-sizing: content`
- `.layout-fs-n` — `field-sizing: none`
- `.layout-c-s` — `contain: strict`

**width 클래스**
퍼센트 기반 (`idx` = 0~9, `weight = (idx+1) * 10`): `.layout-w-${weight}` — `width: ${weight}%; max-width: ${weight}%`  
픽셀 기반 (`idx` = 0~15): `.layout-wp-${idx}` — `width: ${idx * 4}px`

**height 클래스**
퍼센트 기반 (`idx` = 0~9): `.layout-h-${height}` — `height: ${height}%`  
픽셀 기반 (`idx` = 0~15): `.layout-hp-${idx}` — `height: ${idx * 4}px`

**특수 클래스**
- `.layout-el-cv` — 자식 `img`, `video` 요소에 `width: 100%; height: 100%; object-fit: cover; margin: 0` 적용 (커버 레이아웃)
- `.layout-ar-sq` — `aspect-ratio: 1 / 1` (정사각형 비율)
- `.layout-ex-fb` — `--padding` 변수를 이용해 부모 컨테이너 패딩을 음수 마진으로 상쇄하고 너비/높이를 패딩만큼 확장: `margin: calc(var(--padding) * -1) 0 0 calc(var(--padding) * -1)`, `width: calc(100% + var(--padding) * 2)`, `height: calc(100% + var(--padding) * 2)`

## 동작 흐름

모듈 로드 시 `grid`(=4)를 import한 뒤, `layout` 문자열 상수가 여러 배열 생성 및 `map`/`join` 패턴으로 즉시 평가된다. 생성된 문자열은 `index.ts`의 `structuralStyles`에 통합된다.
