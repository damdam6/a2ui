# renderers/react/src/v0_9/catalog/basic/components/Video.tsx

## 개요

`Video` 컴포넌트는 a2ui basic catalog의 `VideoApi` 스키마를 따르는 React 구현체다. `props.url`을 소스로 하는 HTML5 `<video>` 요소를 렌더링하며, 브라우저 기본 컨트롤을 항상 표시한다. 스타일은 `getBaseLeafStyle()`과 CSS 변수로 구성된다.

## 의존성

### 외부 패키지
- `react` — `React`, `React.CSSProperties`
- `@a2ui/web_core/v0_9/basic_catalog` — `VideoApi`

### 저장소 내부 모듈
- [`../../../adapter`](../../../adapter.tsx.md) — `createComponentImplementation`
- [`../utils`](../utils.ts.md) — `getBaseLeafStyle`, `useBasicCatalogStyles`

## Exports

| 이름 | 종류 |
|------|------|
| `Video` | 상수 (ReactComponentImplementation) |

## 상세 명세

### `Video`

`createComponentImplementation(VideoApi, renderFn)` 호출로 생성된 컴포넌트 구현체다.

**렌더 함수 시그니처:** `({props}) => JSX.Element`

**매개변수 (props 필드):**
- `props.url` — 비디오 파일 URL 문자열

**동작 로직:**
1. `useBasicCatalogStyles()`로 global 스타일 주입.
2. `style` 객체 구성:
   - `getBaseLeafStyle()` 스프레드 → `{ boxSizing: 'border-box' }`
   - `width: '100%'`
   - `height: 'auto'`
   - `borderRadius`: CSS 변수 `--a2ui-video-border-radius`(기본값 `0`)
3. `<video src={props.url} controls style={style} />` 반환. `controls` 속성으로 브라우저 기본 재생 컨트롤을 활성화한다.

## 동작 흐름

마운트 시 스타일이 주입되고, `props.url`이 설정된 비디오 요소가 렌더링된다. `props.url` 변경 시 브라우저가 새 소스를 로드한다. 재생·정지·탐색 등의 미디어 컨트롤은 모두 브라우저 네이티브 동작에 위임된다.
