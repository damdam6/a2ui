# renderers/react/src/v0_8/components/layout/List.tsx

## 개요

`List` 컴포넌트는 스크롤 가능한 목록 형태로 자식 컴포넌트들을 렌더링하는 레이아웃 컨테이너다. `direction` 속성으로 수직/수평 방향을 제어하며, Lit 렌더러의 `List` 커스텀 엘리먼트와 동일한 DOM 구조를 유지한다. 스타일은 theme 클래스와 `data-direction` 어트리뷰트를 통해 CSS로 적용된다.

## 의존성

### 외부 패키지
- `react`: `memo`
- `@a2ui/web_core/types/types`: `Types.ListNode`, `Types.AnyComponentNode` (타입 전용 import)

### 저장소 내부 모듈
- [`../../types`](../../types.ts.md): `A2UIComponentProps` 타입
- [`../../hooks/useA2UIComponent`](../../hooks/useA2UIComponent.ts.md): `useA2UIComponent` 훅
- [`../../lib/utils`](../../lib/utils.ts.md): `classMapToString`, `stylesToObject` 유틸
- [`../../core/ComponentNode`](../../core/ComponentNode.tsx.md): `ComponentNode` 재귀 렌더러

## Exports

- `List` — memo로 감싼 named export (React 함수 컴포넌트)
- `default` — `List`를 default export로도 제공

## 상세 명세

### `List` (컴포넌트)

**시그니처:** `memo(function List({ node, surfaceId }: A2UIComponentProps<Types.ListNode>): JSX.Element)`

**내부 초기화:**

1. `useA2UIComponent(node, surfaceId)`로 `theme`을 가져온다.
2. `props.direction ?? 'vertical'`로 방향을 결정한다 (Lit 기본값 `'vertical'`에 맞춤).
3. `Array.isArray(props.children) ? props.children : []`로 자식 배열을 안전하게 추출한다.
4. `node.weight`가 정의된 경우 `{'--weight': node.weight}` CSS 변수 객체를 `hostStyle`로 생성한다.

**자식 렌더링 로직:**

`Card`/`Column`과 동일한 방식으로 `children`을 map하여 `childId`와 `childNode`를 결정한 뒤 `<ComponentNode>`에 위임한다.

**렌더링 구조:**

최상위 `<div className="a2ui-list">`에 `data-direction={direction}`과 `hostStyle`을 적용한다. 내부 `<section>`에 `classMapToString(theme.components.List)` 클래스와 `stylesToObject(theme.additionalStyles?.List)` 스타일을 적용하고 자식들을 렌더링한다.

## 동작 흐름

`direction` 값을 `data-direction` 어트리뷰트로 노출하여 CSS가 세로 또는 가로 스크롤 레이아웃을 적용한다. Column/Row와 달리 `alignment`/`distribution` 속성은 없고 방향 하나만 제어한다. 자식 렌더링은 `ComponentNode`에 위임하여 재귀 처리된다.
