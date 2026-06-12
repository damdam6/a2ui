# renderers/react/src/v0_9/catalog/basic/components/Image.tsx

## 개요

`Image` 컴포넌트는 a2ui Basic Catalog의 이미지 UI 요소다. `ImageApi` 스키마를 기반으로 생성되며, `variant` prop에 따라 `icon`, `avatar`, `smallFeature`, `largeFeature`, `header` 등 다섯 가지 사전 정의된 스타일을 적용한다. `fit` prop은 CSS `object-fit` 속성으로 매핑되어 이미지 배율 방식을 제어한다.

## 의존성

### 외부 패키지
- `react` — `React.CSSProperties` 타입 사용
- `@a2ui/web_core/v0_9/basic_catalog` — `ImageApi` 스키마 가져오기

### 저장소 내부 모듈
- [`../../../adapter`](../../../adapter.tsx.md) — `createComponentImplementation` 유틸리티
- [`../utils`](../utils.ts.md) — `getBaseLeafStyle`, `getWeightStyle`, `useBasicCatalogStyles`

## Exports

| 이름 | 종류 |
|------|------|
| `Image` | 상수 (React 컴포넌트, `createComponentImplementation` 반환값) |

## 상세 명세

### `Image`

**시그니처**: `createComponentImplementation(ImageApi, ({props}) => JSX.Element)`

**렌더 함수 내 지역 함수**:

#### `mapFit(fit?: string): React.CSSProperties['objectFit']`
- `fit === 'scaleDown'`이면 `'scale-down'`을 반환한다 (CSS 값과 API 값의 명칭 불일치 처리).
- 그 외이면 `fit`을 `React.CSSProperties['objectFit']`으로 캐스팅하여 그대로 반환하거나, undefined이면 `'fill'`을 반환한다.

**`style` 객체 구성 로직** (스프레드 순서):
1. `getBaseLeafStyle()` — `{boxSizing: 'border-box'}`
2. `getWeightStyle(props.weight)` — weight가 숫자이면 flex 속성 추가
3. `objectFit: mapFit(props.fit)`
4. `display: 'block'`
5. `borderRadius: 'var(--a2ui-image-border-radius, 0)'`

**`variant` 분기 로직** (style 객체를 직접 변형):
- `'icon'`: `width = 'var(--a2ui-image-icon-size, 24px)'`, `height = 'var(--a2ui-image-icon-size, 24px)'`
- `'avatar'`: `width = 'var(--a2ui-image-avatar-size, 40px)'`, `height = 'var(--a2ui-image-avatar-size, 40px)'`, `borderRadius = '50%'` (원형으로 오버라이드)
- `'smallFeature'`: `maxWidth = 'var(--a2ui-image-small-feature-size, 100px)'`
- `'largeFeature'`: `maxHeight = 'var(--a2ui-image-large-feature-size, 400px)'`
- `'header'`: `height = 'var(--a2ui-image-header-size, 200px)'`, `objectFit = 'cover'` (헤더는 항상 `cover`)
- 일치하는 variant 없음: 스타일 변형 없음

**렌더링**: `<img src={props.url} alt={props.description || ''} style={style} />`

**props**:
- `props.url` (`string`) — 이미지 URL
- `props.description` (`string | undefined`) — alt 텍스트 (없으면 빈 문자열)
- `props.fit` (`string | undefined`) — object-fit 값 (`'scaleDown'` 포함)
- `props.variant` (`'icon' | 'avatar' | 'smallFeature' | 'largeFeature' | 'header' | undefined`) — 크기/모양 변형
- `props.weight` (`number | undefined`) — flex 가중치

## 동작 흐름

스타일을 순차적으로 조립한다: 기본 leaf 스타일 → weight 스타일 → fit 매핑 → variant 분기. `'avatar'`의 경우 이전에 설정된 `borderRadius`를 `'50%'`로 덮어쓰는 점에 주의. `'header'`의 경우 사용자의 `fit` 설정을 무시하고 `'cover'`로 강제한다. 내부 상태 없음.
