# renderers/react/src/v0_9/catalog/basic/components/Divider.tsx

## 개요

`Divider` 컴포넌트는 a2ui Basic Catalog의 구분선 UI 요소다. `DividerApi` 스키마를 기반으로 생성되며, `axis` prop에 따라 수평 또는 수직 방향의 구분선을 렌더링한다. 얇은 `<div>` 엘리먼트에 배경색을 지정하는 방식으로 구현되어 있다.

## 의존성

### 외부 패키지
- `react` — `React.CSSProperties` 타입 사용
- `@a2ui/web_core/v0_9/basic_catalog` — `DividerApi` 스키마 가져오기

### 저장소 내부 모듈
- [`../../../adapter`](../../../adapter.tsx.md) — `createComponentImplementation` 유틸리티
- [`../utils`](../utils.ts.md) — `useBasicCatalogStyles` 훅

## Exports

| 이름 | 종류 |
|------|------|
| `Divider` | 상수 (React 컴포넌트, `createComponentImplementation` 반환값) |

## 상세 명세

### `Divider`

**시그니처**: `createComponentImplementation(DividerApi, ({props}) => JSX.Element)`

**렌더 함수 동작 로직**:

1. `useBasicCatalogStyles()` 호출.
2. `isVertical`: `props.axis === 'vertical'` 여부.
3. `style` 객체 기본값:
   - `border: 'none'` — 기본 border 제거
   - `backgroundColor: 'var(--a2ui-color-border, #ccc)'`
4. `isVertical`에 따라 분기하여 크기 및 margin 설정:
   - **수직 (`isVertical === true`)**:
     - `width: 'var(--a2ui-border-width, 1px)'`
     - `height: '100%'`
     - `margin: '0 var(--a2ui-divider-spacing, var(--a2ui-spacing-m, 0.5rem))'` — 좌우 여백
   - **수평 (`isVertical === false`)**:
     - `width: '100%'`
     - `height: 'var(--a2ui-border-width, 1px)'`
     - `margin: 'var(--a2ui-divider-spacing, var(--a2ui-spacing-m, 0.5rem)) 0'` — 상하 여백
5. `<div style={style} />` 를 반환한다.

**props**:
- `props.axis` (`'vertical' | 'horizontal' | undefined`) — 구분선 방향. `'vertical'`이 아닌 모든 값은 수평으로 처리된다.

## 동작 흐름

props를 받아 수직/수평 여부를 결정하고, 그에 맞게 `style` 객체를 구성한 뒤 단일 `<div>`를 반환한다. 내부 상태 없음. `border: none` 과 `backgroundColor`를 조합하여 구분선 효과를 구현하므로 HTML `<hr>` 대신 `<div>` 를 사용한다.
