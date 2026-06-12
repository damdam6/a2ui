# renderers/react/src/v0_8/components/layout/Column.tsx

## 개요

`Column` 컴포넌트는 자식 컴포넌트들을 flexbox 세로 방향으로 배치하는 레이아웃 컨테이너다. `alignment`(교차축 정렬)과 `distribution`(주축 분배) 속성을 `data-*` 어트리뷰트로 루트 요소에 전달하며, CSS가 이를 통해 스타일을 적용한다. Lit 렌더러의 `Column` 커스텀 엘리먼트와 동일한 구조를 유지한다.

## 의존성

### 외부 패키지
- `react`: `memo`
- `@a2ui/web_core/types/types`: `Types.ColumnNode`, `Types.AnyComponentNode` (타입 전용 import)

### 저장소 내부 모듈
- [`../../types`](../../types.ts.md): `A2UIComponentProps` 타입
- [`../../hooks/useA2UIComponent`](../../hooks/useA2UIComponent.ts.md): `useA2UIComponent` 훅
- [`../../lib/utils`](../../lib/utils.ts.md): `classMapToString`, `stylesToObject` 유틸
- [`../../core/ComponentNode`](../../core/ComponentNode.tsx.md): `ComponentNode` 재귀 렌더러

## Exports

- `Column` — memo로 감싼 named export (React 함수 컴포넌트)
- `default` — `Column`을 default export로도 제공

## 상세 명세

### `Column` (컴포넌트)

**시그니처:** `memo(function Column({ node, surfaceId }: A2UIComponentProps<Types.ColumnNode>): JSX.Element)`

**내부 초기화:**

1. `useA2UIComponent(node, surfaceId)`로 `theme`을 가져온다.
2. `props.alignment ?? 'stretch'`로 교차축 정렬을 결정한다 (Lit 기본값 `'stretch'`에 맞춤).
3. `props.distribution ?? 'start'`로 주축 분배를 결정한다 (Lit 기본값 `'start'`에 맞춤).
4. `Array.isArray(props.children) ? props.children : []`로 자식 배열을 안전하게 추출한다.
5. `node.weight`가 정의된 경우 `{'--weight': node.weight}` CSS 변수 객체를 `hostStyle`로 생성한다.

**자식 렌더링 로직:**

`Card`와 동일한 방식으로 `children`을 map하여 `childId`(`'id'` 속성 있으면 해당 id, 없으면 `child-${index}`)와 `childNode`(`'type'` 속성 있으면 `Types.AnyComponentNode`로 캐스팅, 없으면 `null`)를 결정한 뒤 `<ComponentNode>`에 위임한다.

**렌더링 구조:**

최상위 `<div className="a2ui-column">`에 `data-alignment={alignment}`, `data-distribution={distribution}`, `hostStyle`을 적용한다. 내부 `<section>`에 `classMapToString(theme.components.Column)` 클래스와 `stylesToObject(theme.additionalStyles?.Column)` 스타일을 적용하고 자식들을 렌더링한다.

## 동작 흐름

`alignment`와 `distribution` 값을 `data-*` 어트리뷰트로 노출하여 CSS가 이를 읽어 세로 flex 레이아웃을 제어하도록 한다. 자식 렌더링은 `ComponentNode`에 위임하여 재귀 처리된다.
