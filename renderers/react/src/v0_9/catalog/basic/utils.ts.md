# renderers/react/src/v0_9/catalog/basic/utils.ts

## 개요

v0_9 basic catalog 컴포넌트들이 공통으로 사용하는 유틸리티 hook과 스타일 변환 함수 모음이다. CSS 변수 스타일 주입, flex 정렬 문자열 매핑, 기본 박스 모델 스타일, flex weight 스타일 계산 기능을 제공한다. 모든 컴포넌트가 이 파일을 import하므로 스타일 일관성의 핵심 지점이 된다.

## 의존성

### 외부 패키지
- `react` — `React.CSSProperties` (타입 import), `useEffect`
- `@a2ui/web_core/v0_9/basic_catalog` — `injectBasicCatalogStyles`

### 저장소 내부 모듈
없음

## Exports

| 이름 | 종류 |
|------|------|
| `useBasicCatalogStyles` | 상수 (함수, React hook) |
| `mapJustify` | 상수 (함수) |
| `mapAlign` | 상수 (함수) |
| `getBaseLeafStyle` | 상수 (함수) |
| `getBaseContainerStyle` | 상수 (함수) |
| `getWeightStyle` | 상수 (함수) |

## 상세 명세

### `useBasicCatalogStyles`

**시그니처:** `() => void`

`useEffect`를 사용하여 컴포넌트 마운트 시 global CSS 변수를 DOM에 주입하는 hook이다. 의존성 배열이 `[]`이므로 컴포넌트 생애 주기 중 정확히 한 번만 실행된다. 내부 로직: `typeof document !== 'undefined' && document.adoptedStyleSheets`가 truthy일 때만 `injectBasicCatalogStyles()`를 호출한다. SSR 환경처럼 `document`가 없는 경우를 안전하게 처리한다.

### `mapJustify`

**시그니처:** `(j?: string) => string`

a2ui `justify` prop 값을 CSS `justify-content` 값으로 변환한다.

| 입력 | 반환 |
|------|------|
| `'center'` | `'center'` |
| `'end'` | `'flex-end'` |
| `'spaceAround'` | `'space-around'` |
| `'spaceBetween'` | `'space-between'` |
| `'spaceEvenly'` | `'space-evenly'` |
| `'start'` | `'flex-start'` |
| `'stretch'` | `'stretch'` |
| 그 외 / `undefined` | `'flex-start'` (기본값) |

### `mapAlign`

**시그니처:** `(a?: string) => string`

a2ui `align` prop 값을 CSS `align-items` 값으로 변환한다.

| 입력 | 반환 |
|------|------|
| `'start'` | `'flex-start'` |
| `'center'` | `'center'` |
| `'end'` | `'flex-end'` |
| `'stretch'` | `'stretch'` |
| 그 외 / `undefined` | `'stretch'` (기본값) |

`mapJustify`와 달리 기본값이 `'stretch'`임에 유의한다.

### `getBaseLeafStyle`

**시그니처:** `() => React.CSSProperties`

리프(leaf) 컴포넌트용 기본 스타일 객체를 반환한다. 반환값: `{ boxSizing: 'border-box' }`.

### `getBaseContainerStyle`

**시그니처:** `() => React.CSSProperties`

컨테이너 컴포넌트용 기본 스타일 객체를 반환한다. 반환값: `{ boxSizing: 'border-box' }`. 현재는 `getBaseLeafStyle`과 동일한 값이지만 의미적으로 구분되어 있다.

### `getWeightStyle`

**시그니처:** `(weight?: number) => React.CSSProperties`

flex 레이아웃에서 자식 컴포넌트의 크기 비율을 결정하는 스타일 객체를 반환한다.

- `weight`가 `number` 타입이 아니면 빈 객체 `{}`를 반환한다.
- `weight`가 숫자이면 `{ flex: '${weight}', minWidth: 0, minHeight: 0 }`을 반환한다.
- `minWidth: 0`과 `minHeight: 0`은 내용이 클 때 flex 컨테이너가 overflow되는 것을 방지하기 위해 자식의 내재적 크기 하한을 제거한다.

## 동작 흐름

컴포넌트 렌더링 시 `useBasicCatalogStyles`는 DOM에 CSS 변수 시트를 한 번 주입한다. 이후 `mapJustify`, `mapAlign`은 inline style 객체 구성 시 동기적으로 호출되어 a2ui 명세 값을 CSS 값으로 변환한다. `getWeightStyle`은 weight prop이 있는 컴포넌트가 flex 레이아웃에서 올바르게 늘어나고 줄어들 수 있도록 한다.
