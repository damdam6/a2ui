# renderers/web_core/src/v0_8/styles/icons.ts

## 개요

Google Material Symbols / Google Symbols 아이콘 폰트를 위한 CSS 클래스 정의를 문자열 상수로 export한다. HTML 요소에 `g-icon` 클래스를 부여하고 텍스트 내용으로 아이콘 이름을 넣으면 아이콘이 렌더링된다. 의존성 없이 순수 CSS 문자열만 제공하는 간단한 파일이다.

## 의존성

없음

## Exports

| 이름 | 종류 |
|------|------|
| `icons` | 상수 (`string`) |

## 상세 명세

### `icons`

export되는 CSS 문자열 상수. `.g-icon` 선택자 블록 하나와 그 안의 두 변형 클래스로 구성된다.

**`.g-icon` 기본 스타일**:
- `font-family: "Material Symbols Outlined", "Google Symbols"` — 아이콘 폰트 지정
- `font-weight: normal; font-style: normal` — 폰트 스타일 초기화
- `font-display: optional` — 폰트 로딩 전략
- `font-size: 24px` — 기본 크기
- `width: 1em; height: 1em` — 크기를 font-size에 비례
- `user-select: none` — 텍스트 선택 방지
- `line-height: 1; letter-spacing: normal; text-transform: none`
- `display: inline-block; white-space: nowrap; word-wrap: normal`
- `direction: ltr`
- `font-feature-settings: "liga"; -webkit-font-feature-settings: "liga"` — 리거처 활성화
- `-webkit-font-smoothing: antialiased; text-rendering: optimizeLegibility; -moz-osx-font-smoothing: grayscale`
- `overflow: hidden`
- `font-variation-settings: "FILL" 0, "wght" 300, "GRAD" 0, "opsz" 48, "ROND" 100` — 기본 변형: 아웃라인, 300 웨이트, 48 크기

**`.g-icon.filled` 중첩 선택자**:
- `font-variation-settings: "FILL" 1, "wght" 300, "GRAD" 0, "opsz" 48, "ROND" 100` — 채워진(filled) 아이콘 변형

**`.g-icon.filled-heavy` 중첩 선택자**:
- `font-variation-settings: "FILL" 1, "wght" 700, "GRAD" 0, "opsz" 48, "ROND" 100` — 채워진 + 굵은(700 웨이트) 아이콘 변형

**사용 예**: `<span class="g-icon">pen_spark</span>`

## 동작 흐름

모듈 로드 시 `icons` 문자열 상수가 즉시 평가된다. `index.ts`에서 import되어 `structuralStyles` 합산에 포함된다.
