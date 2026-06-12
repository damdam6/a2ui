# renderers/react/src/v0_8/components/layout/Row.tsx

## 개요

`Row` 컴포넌트는 자식 컴포넌트들을 flexbox 가로 방향으로 배치하는 레이아웃 컨테이너다. `alignment`(교차축 정렬)과 `distribution`(주축 분배) 속성을 `data-*` 어트리뷰트로 루트 요소에 전달하며, CSS가 이를 통해 스타일을 적용한다. `Column`과 대칭 구조를 가지며 Lit 렌더러의 `Row` 커스텀 엘리먼트와 동일한 DOM 구조를 유지한다.

## 의존성

### 외부 패키지
- `react`: `memo`
- `@a2ui/web_core/types/types`: `Types.RowNode`, `Types.AnyComponentNode` (타입 전용 import)

### 저장소 내부 모듈
- [`../../types`](../../types.ts.md): `A2UIComponentProps` 타입
- [`../../hooks/useA2UIComponent`](../../hooks/useA2UIComponent.ts.md): `useA2UIComponent` 훅
- [`../../lib/utils`](../../lib/utils.ts.md): `classMapToString`, `stylesToObject` 유틸
- [`../../core/ComponentNode`](../../core/ComponentNode.tsx.md): `ComponentNode` 재귀 렌더러

## Exports

- `Row` — memo로 감싼 named export (React 함수 컴포넌트)
- `default` — `Row`를 default export로도 제공

## 상세 명세

### `Row` (컴포넌트)

**시그니처:** `memo(function Row({ node, surfaceId }: A2UIComponentProps<Types.RowNode>): JSX.Element)`

**내부 초기화:**

1. `useA2UIComponent(node, surfaceId)`로 `theme`을 가져온다.
2. `props.alignment ?? 'stretch'`로 교차축 정렬을 결정한다 (Lit 기본값 `'stretch'`에 맞춤).
3. `props.distribution ?? 'start'`로 주축 분배를 결정한다 (Lit 기본값 `'start'`에 맞춤).
4. `Array.isArray(props.children) ? props.children : []`로 자식 배열을 안전하게 추출한다.
5. `node.weight`가 정의된 경우 `{'--weight': node.weight}` CSS 변수 객체를 `hostStyle`로 생성한다.

**자식 렌더링 로직:**

`Column`과 동일한 방식으로 `children`을 map하여 `childId`와 `childNode`를 결정한 뒤 `<ComponentNode>`에 위임한다.

**렌더링 구조:**

최상위 `<div className="a2ui-row">`에 `data-alignment={alignment}`, `data-distribution={distribution}`, `hostStyle`을 적용한다. 내부 `<section>`에 `classMapToString(theme.components.Row)` 클래스와 `stylesToObject(theme.additionalStyles?.Row)` 스타일을 적용하고 자식들을 렌더링한다.

## 동작 흐름

`Column`과 구조가 동일하며, CSS 레벨에서 `a2ui-row` 클래스로 가로 flex 방향을 적용하는 점만 다르다. `alignment`와 `distribution` 값을 `data-*` 어트리뷰트로 노출하여 CSS가 세부 정렬을 제어한다.
