# renderers/web_core/src/v0_8/styles/border.ts

## 개요

테두리(border) 너비, 외곽선(outline) 너비, 테두리 반경(border-radius), 테두리 스타일에 관한 CSS 유틸리티 클래스들을 문자열로 생성하여 export한다. 픽셀 값은 `shared.ts`의 `grid` 상수(4px)를 기반으로 계산된다.

## 의존성

### 저장소 내부 모듈
- [`./shared.ts`](./shared.ts.md) — `grid` (값: `4`)

## Exports

| 이름 | 종류 |
|------|------|
| `border` | 상수 (`string`) |

## 상세 명세

### `border`

export되는 CSS 문자열 상수. `new Array(25).fill(0).map((_, idx) => ...)` 패턴으로 `idx` = 0~24에 대해 각 인덱스마다 다음 7개의 클래스를 생성한다:

- `.border-bw-${idx}` — `border-width: ${idx}px` (전체 테두리 너비)
- `.border-btw-${idx}` — `border-top-width: ${idx}px`
- `.border-bbw-${idx}` — `border-bottom-width: ${idx}px`
- `.border-blw-${idx}` — `border-left-width: ${idx}px`
- `.border-brw-${idx}` — `border-right-width: ${idx}px`
- `.border-ow-${idx}` — `outline-width: ${idx}px`
- `.border-br-${idx}` — `border-radius: ${idx * grid}px; overflow: hidden` (border-radius는 grid 단위로 계산, 즉 `idx * 4px`)

추가로 다음 두 클래스가 포함된다:
- `.border-br-50pc` — `border-radius: 50%` (완전한 원형)
- `.border-bs-s` — `border-style: solid`

## 동작 흐름

모듈 로드 시 `grid`(=4)를 import한 뒤, 배열 생성 및 `map`/`join`으로 CSS 문자열이 즉시 평가된다. `border-br-${idx}` 클래스의 `border-radius` 값은 `idx * 4px` 이지만 `border-width` 계열은 `idx`를 그대로 픽셀로 사용한다. 생성된 문자열은 `index.ts`의 `structuralStyles`에 통합된다.
