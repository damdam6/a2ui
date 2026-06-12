# renderers/web_core/src/v0_9/basic_catalog/styles/default.ts

## 개요

A2UI 기본 카탈로그의 기본 테마 CSS 변수 정의와, 그것을 문서에 주입하는 유틸리티를 제공하는 파일이다. `DEFAULT_CSS` 문자열 상수에 모든 디자인 토큰(색상, 간격, 폰트 크기 등)이 CSS 커스텀 프로퍼티로 선언되어 있으며, `injectBasicCatalogStyles()`가 이를 `document.adoptedStyleSheets`에 한 번만 추가한다. `computeColorVariant()`는 CSS `color-mix()` 및 `light-dark()` 함수 문자열을 생성하는 순수 함수로, `DEFAULT_CSS` 내부에서 직접 호출되어 변수 값을 생성한다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈
없음

## Exports

| 이름 | 종류 | 설명 |
|---|---|---|
| `injectBasicCatalogStyles` | 함수 | 기본 테마 CSS를 `document.adoptedStyleSheets`에 주입 |
| `computeColorVariant` | 함수(오버로드) | 색상 변형(light/dark/hover) CSS 공식 문자열 생성 |
| `ColorVariantLightDarkOptions` | 인터페이스 | light/dark 변형 옵션 타입 |
| `ColorVariantHoverOptions` | 인터페이스 | hover 변형 옵션 타입 |

## 상세 명세

### 비공개 상수: `DEFAULT_CSS: string`

A2UI 기본 테마의 전체 CSS 마크업을 담은 템플릿 문자열이다. `:where()` 선택자를 사용해 특이성(specificity)을 0으로 유지하므로, 소비자의 페이지 스타일이 쉽게 덮어쓸 수 있다.

정의하는 CSS 커스텀 프로퍼티:

- **색상 체계**: `:where(:root)`에 `color-scheme: light dark` 설정. `.a2ui-dark` 클래스는 dark 강제, `.a2ui-light` 클래스는 light 강제
- **배경/표면**: `--a2ui-color-background` (`light-dark(#eee, #111)`), `--a2ui-color-on-background` (`light-dark(#333, #eee)`). `--a2ui-color-surface`는 배경색에서 흰색을 85%(light) 또는 95%(dark) 비율로 혼합한 값
- **주 색상**: `--a2ui-color-primary: #17e`, light/dark/hover 변형은 `computeColorVariant` 호출로 생성. `--a2ui-color-on-primary: #fff`
- **보조 색상**: `--a2ui-color-secondary` (`light-dark(#ddd, #333)`), light 변형(기본 85%), dark 변형(95%), hover 변형. `--a2ui-color-on-secondary: light-dark(#333, #eee)`
- **테두리**: `--a2ui-border-radius: 0.25rem`, `--a2ui-color-border: light-dark(#ccc, #444)`, `--a2ui-border-width: 1px`, `--a2ui-border: 1px solid var(--a2ui-color-border, #ccc)`
- **폰트**: `--a2ui-font-family-title: inherit`, `--a2ui-font-family-monospace: monospace`, `--a2ui-color-input: light-dark(#fff, #2a2a2a)`, `--a2ui-color-on-input: light-dark(#333, #eee)`
- **간격**: 기준 `--a2ui-grid-base: 0.5rem`. xs = s/2, s = m/2, m = grid-base, l = m×2, xl = l×2
- **폰트 크기**: 기준 `--a2ui-font-size: 1rem`, 배율 `--a2ui-font-scale: 1.2`. xs = s/scale, s = m/scale, m = font-size, l = m×scale, xl = l×scale, 2xl = xl×scale
- **줄 높이**: `--a2ui-line-height-headings: 1.2`, `--a2ui-line-height-body: 1.5`

---

### 비공개 변수: `defaultStyleSheet: CSSStyleSheet | undefined`

`getDefaultStyleSheet()` 의 결과를 캐싱하는 모듈 수준 변수. 최초 호출 전에는 `undefined`.

---

### 비공개 함수: `getDefaultStyleSheet(): CSSStyleSheet`

`defaultStyleSheet`가 `undefined`이면 새 `CSSStyleSheet`를 생성하고 `DEFAULT_CSS`로 초기화(`replaceSync`)한 뒤 캐싱한다. 이후 호출은 캐시된 인스턴스를 바로 반환한다.

---

### `injectBasicCatalogStyles(): void`

- 매개변수: 없음
- 반환: `void`
- `document`가 `undefined`이면(SSR 환경) 즉시 반환한다.
- `getDefaultStyleSheet()`로 스타일시트를 가져온 뒤, `document.adoptedStyleSheets`에 이미 포함되어 있지 않을 때만 스프레드 연산으로 추가한다. 따라서 중복 주입이 방지된다.
- `@a2ui/lit`, `@a2ui/angular`, `@a2ui/react`의 기본 카탈로그 패키지들이 이 함수를 호출해 디자인 토큰을 공유한다. A2UI 스펙의 일부가 아닌 편의 기능이다.

---

### 인터페이스: `ColorVariantLightDarkOptions`

light 또는 dark 변형을 계산할 때 사용하는 옵션.

| 필드 | 타입 | 설명 |
|---|---|---|
| `colorVar` | `string` | 기준 색상의 CSS 변수명 (예: `'--a2ui-color-primary'`) |
| `percentage` | `number` (optional) | 기준 색상이 차지할 비율 (기본값 `85`) |
| `mixColor` | `string` (optional) | 혼합할 색상 (light 기본 `'white'`, dark 기본 `'black'`) |

---

### 인터페이스: `ColorVariantHoverOptions`

hover 변형을 계산할 때 사용하는 옵션.

| 필드 | 타입 | 설명 |
|---|---|---|
| `darkVar` | `string` | dark 변형 CSS 변수명 |
| `lightVar` | `string` | light 변형 CSS 변수명 |

---

### `computeColorVariant` (오버로드 3개)

**시그니처 1**: `computeColorVariant(type: 'light' | 'dark', options: ColorVariantLightDarkOptions): string`

**시그니처 2**: `computeColorVariant(type: 'hover', options: ColorVariantHoverOptions): string`

**구현 시그니처**: `computeColorVariant(type: 'light' | 'dark' | 'hover', options: ColorVariantLightDarkOptions | ColorVariantHoverOptions): string`

동작:
- `type === 'light'`: `color-mix(in oklab, var(<colorVar>) <percentage>%, <mixColor>)` 형태의 문자열 반환. `percentage` 기본값 `85`, `mixColor` 기본값 `'white'`
- `type === 'dark'`: 위와 동일하나 `mixColor` 기본값이 `'black'`
- `type === 'hover'`: `light-dark(var(<darkVar>), var(<lightVar>))` 형태의 문자열 반환. 라이트 모드에서 dark 변형을, 다크 모드에서 light 변형을 사용하는 역전 패턴임에 유의

## 동작 흐름

모듈 로드 시 `DEFAULT_CSS` 문자열 리터럴이 평가된다. 이때 `computeColorVariant` 호출이 인라인 템플릿 표현식으로 실행되어 CSS 변수 값이 확정된다. 이후 `injectBasicCatalogStyles()`가 처음 호출될 때 `getDefaultStyleSheet()`가 `CSSStyleSheet`를 생성·캐시하고, 문서에 한 번만 추가한다. 두 번째 이후 호출은 중복 체크 후 무시된다.
