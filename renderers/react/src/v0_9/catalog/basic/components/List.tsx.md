# renderers/react/src/v0_9/catalog/basic/components/List.tsx

## 개요

`List` 컴포넌트는 a2ui Basic Catalog의 스크롤 가능한 방향성 목록 레이아웃 컨테이너다. `ListApi` 스키마를 기반으로 생성되며, `direction` prop에 따라 수평 또는 수직 flex 방향을 설정하고, 각각의 방향에 맞게 overflow 스크롤 설정을 적용한다. 자식 컴포넌트 목록 렌더링은 `ChildList`에 위임한다.

## 의존성

### 외부 패키지
- `react` — `React.CSSProperties` 타입 사용
- `@a2ui/web_core/v0_9/basic_catalog` — `ListApi` 스키마 가져오기

### 저장소 내부 모듈
- [`../../../adapter`](../../../adapter.tsx.md) — `createComponentImplementation` 유틸리티
- [`./ChildList`](./ChildList.tsx.md) — 자식 목록 렌더링 헬퍼 컴포넌트
- [`../utils`](../utils.ts.md) — `mapAlign`, `useBasicCatalogStyles`

## Exports

| 이름 | 종류 |
|------|------|
| `List` | 상수 (React 컴포넌트, `createComponentImplementation` 반환값) |

## 상세 명세

### `List`

**시그니처**: `createComponentImplementation(ListApi, ({props, buildChild, context}) => JSX.Element)`

**렌더 함수 동작 로직**:

1. `useBasicCatalogStyles()` 호출.
2. `isHorizontal`: `props.direction === 'horizontal'` 여부.
3. `style` 객체 구성:
   - `display: 'flex'`
   - `flexDirection: isHorizontal ? 'row' : 'column'`
   - `alignItems: mapAlign(props.align)` — `start`→`flex-start`, `center`→`center`, `end`→`flex-end`, `stretch`→`stretch`, 기타/undefined→`stretch`
   - `overflowX: isHorizontal ? 'auto' : 'hidden'` — 수평이면 가로 스크롤, 수직이면 가로 숨김
   - `overflowY: isHorizontal ? 'hidden' : 'auto'` — 수평이면 세로 숨김, 수직이면 세로 스크롤
   - `gap: 'var(--a2ui-list-gap, var(--a2ui-spacing-s))'`
   - `padding: 'var(--a2ui-list-padding, 0)'`
4. `<div style={style}>` 안에 `<ChildList childList={props.children} buildChild={buildChild} context={context} />`를 렌더링한다.

**props**:
- `props.direction` (`'horizontal' | 'vertical' | undefined`) — 배치 방향. `'horizontal'`이 아닌 모든 값은 수직으로 처리.
- `props.align` (`string | undefined`) — 교차축 정렬
- `props.children` (`ResolvedChildList`) — 렌더링할 자식 컴포넌트 참조 목록

## 동작 흐름

`direction`에 따라 수평/수직 레이아웃을 결정하고, overflow를 대칭적으로 설정하여 방향에 맞는 스크롤만 활성화한다. 자식 렌더링은 `ChildList`에 위임하며, 내부 상태 없음. `Column`과 유사한 구조이나 `justify` prop 없이 `align`만 지원하고, overflow 스크롤 처리가 추가된 점이 다르다.
