# renderers/react/src/v0_9/catalog/basic/components/Card.tsx

## 개요

`Card` 컴포넌트는 a2ui Basic Catalog의 카드 레이아웃 컨테이너다. `CardApi` 스키마를 기반으로 생성되며, 테두리·둥근 모서리·배경·그림자를 갖는 카드 형태의 `<div>`를 렌더링한다. `weight` prop을 통해 flex 레이아웃에서 차지하는 공간 비율을 제어할 수 있으며, 내부에 단일 자식 컴포넌트를 렌더링한다.

## 의존성

### 외부 패키지
- `react` — `React.CSSProperties` 타입 사용
- `@a2ui/web_core/v0_9/basic_catalog` — `CardApi` 스키마 가져오기

### 저장소 내부 모듈
- [`../../../adapter`](../../../adapter.tsx.md) — `createComponentImplementation` 유틸리티
- [`../utils`](../utils.ts.md) — `getBaseContainerStyle`, `getWeightStyle`, `useBasicCatalogStyles`

## Exports

| 이름 | 종류 |
|------|------|
| `Card` | 상수 (React 컴포넌트, `createComponentImplementation` 반환값) |

## 상세 명세

### `Card`

**시그니처**: `createComponentImplementation(CardApi, ({props, buildChild}) => JSX.Element)`

**렌더 함수 동작 로직**:

1. `useBasicCatalogStyles()`를 호출하여 전역 스타일을 주입한다.
2. `style` 객체를 구성한다 (스프레드 순서):
   - `getBaseContainerStyle()` — `{boxSizing: 'border-box'}`
   - `getWeightStyle(props.weight)` — `weight`가 숫자이면 `{flex: '${weight}', minWidth: 0, minHeight: 0}`, 아니면 `{}`
   - `display: 'block'`
   - `border: 'var(--a2ui-card-border, var(--a2ui-border))'`
   - `borderRadius: 'var(--a2ui-card-border-radius, var(--a2ui-border-radius, 8px))'`
   - `padding: 'var(--a2ui-card-padding, var(--a2ui-spacing-m, 16px))'`
   - `background: 'var(--a2ui-card-background, var(--a2ui-color-surface, #fff))'`
   - `color: 'var(--a2ui-color-on-surface, #333)'`
   - `boxShadow: 'var(--a2ui-card-box-shadow, 0 2px 4px rgba(0,0,0,0.1))'`
   - `margin: 'var(--a2ui-card-margin, var(--a2ui-spacing-m))'`
3. `<div style={style}>` 안에 `props.child`가 존재하면 `buildChild(props.child)`를 렌더링하고, 없으면 `null`을 렌더링한다.

**props**:
- `props.weight` (`number | undefined`) — flex 가중치
- `props.child` (`ComponentId | undefined`) — 카드 내부에 렌더링할 자식 컴포넌트 ID

## 동작 흐름

스타일을 빌드 시점에 인라인으로 계산하고 단일 `<div>` 를 반환한다. 내부 상태 없음. 모든 시각적 속성은 CSS 변수의 fallback 체인으로 정의되어 상위 테마 변수 → 컴포넌트별 변수 → 하드코딩된 기본값 순으로 해석된다.
