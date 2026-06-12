# renderers/react/src/v0_9/catalog/basic/components/Column.tsx

## 개요

`Column` 컴포넌트는 a2ui Basic Catalog의 수직 flex 레이아웃 컨테이너다. `ColumnApi` 스키마를 기반으로 생성되며, 자식 컴포넌트 목록(`props.children`)을 `ChildList`를 통해 수직 방향으로 배치한다. `justify`, `align`, `weight` prop을 통해 정렬 방식과 flex 가중치를 제어한다.

## 의존성

### 외부 패키지
- `@a2ui/web_core/v0_9/basic_catalog` — `ColumnApi` 스키마 가져오기

### 저장소 내부 모듈
- [`../../../adapter`](../../../adapter.tsx.md) — `createComponentImplementation` 유틸리티
- [`./ChildList`](./ChildList.tsx.md) — 자식 목록 렌더링 헬퍼 컴포넌트
- [`../utils`](../utils.ts.md) — `mapJustify`, `mapAlign`, `getWeightStyle`, `useBasicCatalogStyles`

## Exports

| 이름 | 종류 |
|------|------|
| `Column` | 상수 (React 컴포넌트, `createComponentImplementation` 반환값) |

## 상세 명세

### `Column`

**시그니처**: `createComponentImplementation(ColumnApi, ({props, buildChild, context}) => JSX.Element)`

**렌더 함수 동작 로직**:

1. `useBasicCatalogStyles()` 호출.
2. 인라인 `style` 객체를 구성한다 (스프레드 순서):
   - `getWeightStyle(props.weight)` — weight가 숫자이면 `{flex: '${weight}', minWidth: 0, minHeight: 0}`, 아니면 `{}`
   - `display: 'flex'`
   - `flexDirection: 'column'`
   - `justifyContent: mapJustify(props.justify)` — `start`→`flex-start`, `end`→`flex-end`, `center`→`center`, `spaceAround`→`space-around`, `spaceBetween`→`space-between`, `spaceEvenly`→`space-evenly`, `stretch`→`stretch`, 기타/undefined→`flex-start`
   - `alignItems: mapAlign(props.align)` — `start`→`flex-start`, `center`→`center`, `end`→`flex-end`, `stretch`→`stretch`, 기타/undefined→`stretch`
   - `gap: 'var(--a2ui-column-gap, var(--a2ui-spacing-m))'`
3. `<div style={style}>` 안에 `<ChildList childList={props.children} buildChild={buildChild} context={context} />`를 렌더링한다.

**props**:
- `props.weight` (`number | undefined`) — flex 가중치
- `props.justify` (`string | undefined`) — 주축(main axis) 정렬
- `props.align` (`string | undefined`) — 교차축(cross axis) 정렬
- `props.children` (`ResolvedChildList`) — 렌더링할 자식 컴포넌트 참조 목록

## 동작 흐름

`mapJustify`, `mapAlign`, `getWeightStyle`을 통해 스타일을 계산하고, `ChildList`에 자식 목록 렌더링을 위임한다. 내부 상태 없음. `ChildList`가 실제 자식 컴포넌트 빌드를 담당하고, `Column`은 배치 컨테이너 역할만 한다.
