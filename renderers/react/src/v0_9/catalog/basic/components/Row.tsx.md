# renderers/react/src/v0_9/catalog/basic/components/Row.tsx

## 개요

`Row` 컴포넌트는 a2ui basic catalog의 `RowApi` 스키마를 따르는 React 구현체다. 자식 컴포넌트들을 가로(row) 방향 flex 컨테이너로 배치하며, `justify` / `align` / `weight` props에 따라 정렬 방식을 동적으로 결정한다. 내부적으로 `ChildList`를 사용하여 자식 목록을 렌더링한다.

## 의존성

### 외부 패키지
- `@a2ui/web_core/v0_9/basic_catalog` — `RowApi`

### 저장소 내부 모듈
- [`../../../adapter`](../../../adapter.tsx.md) — `createComponentImplementation`
- [`./ChildList`](./ChildList.tsx.md) — `ChildList`
- [`../utils`](../utils.ts.md) — `mapJustify`, `mapAlign`, `getWeightStyle`, `useBasicCatalogStyles`

## Exports

| 이름 | 종류 |
|------|------|
| `Row` | 상수 (ReactComponentImplementation) |

## 상세 명세

### `Row`

`createComponentImplementation(RowApi, renderFn)` 호출로 생성된 컴포넌트 구현체다.

**렌더 함수 시그니처:** `({props, buildChild, context}) => JSX.Element`

**매개변수 (props 필드):**
- `props.justify` — 주축(가로) 정렬 지시어 (옵셔널 문자열). `mapJustify`에 의해 CSS `justify-content` 값으로 변환됨
- `props.align` — 교차축(세로) 정렬 지시어 (옵셔널 문자열). `mapAlign`에 의해 CSS `align-items` 값으로 변환됨
- `props.weight` — flex grow/shrink 비율 (옵셔널 숫자). `getWeightStyle`에 의해 처리됨
- `props.children` — 자식 참조 목록 (`ResolvedChildList`)
- `buildChild` — 자식 ID를 React 노드로 변환하는 함수
- `context` — `ComponentContext` 객체. `ChildList`에 그대로 전달됨

**동작 로직:**

1. `useBasicCatalogStyles()`로 global 스타일을 DOM에 주입한다.
2. `display: 'flex'`, `flexDirection: 'row'`인 `<div>`를 반환한다.
3. `justifyContent`는 `mapJustify(props.justify)` 결과를 사용한다.
4. `alignItems`는 `mapAlign(props.align)` 결과를 사용한다.
5. `gap`은 CSS 변수 `'var(--a2ui-row-gap, var(--a2ui-spacing-m))'`를 사용한다.
6. `getWeightStyle(props.weight)` 스타일을 스프레드 연산자로 앞에 병합한다.
7. `<ChildList childList={props.children} buildChild={buildChild} context={context} />`로 자식들을 렌더링한다.

## 동작 흐름

props가 업데이트되면 `mapJustify` / `mapAlign` / `getWeightStyle` 변환 함수들이 새 정렬값·크기값을 계산하고, `ChildList`가 `props.children` 배열을 순회하여 각 자식을 렌더링한다. flex 레이아웃 특성상 자식들은 기본적으로 가로로 나열되며, gap 변수로 간격이 조정된다.
