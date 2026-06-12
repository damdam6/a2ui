# renderers/react/src/v0_8/components/layout/Card.tsx

## 개요

`Card` 컴포넌트는 콘텐츠를 시각적으로 그룹화하는 컨테이너 역할을 한다. Lit의 `Card` 커스텀 엘리먼트와 동일한 DOM 구조로 자식 컴포넌트 노드들을 재귀적으로 렌더링하며, 테마 시스템에서 제공하는 클래스(border, padding, background 등)를 적용한다. 인라인 스타일 대신 theme CSS 클래스로 스타일을 전달하는 구조다.

## 의존성

### 외부 패키지
- `react`: `memo`
- `@a2ui/web_core/types/types`: `Types.CardNode`, `Types.AnyComponentNode` (타입 전용 import)

### 저장소 내부 모듈
- [`../../types`](../../types.ts.md): `A2UIComponentProps` 타입
- [`../../hooks/useA2UIComponent`](../../hooks/useA2UIComponent.ts.md): `useA2UIComponent` 훅
- [`../../lib/utils`](../../lib/utils.ts.md): `classMapToString`, `stylesToObject` 유틸
- [`../../core/ComponentNode`](../../core/ComponentNode.tsx.md): `ComponentNode` 재귀 렌더러

## Exports

- `Card` — memo로 감싼 named export (React 함수 컴포넌트)
- `default` — `Card`를 default export로도 제공

## 상세 명세

### `Card` (컴포넌트)

**시그니처:** `memo(function Card({ node, surfaceId }: A2UIComponentProps<Types.CardNode>): JSX.Element)`

**내부 초기화:**

1. `useA2UIComponent(node, surfaceId)`로 `theme`을 가져온다 (다른 반환값은 사용하지 않음).
2. 자식 노드 정규화: `props.children`이 있으면 사용하고, 없으면 `props.child`를 단일 항목 배열 `[props.child]`로 감싼다. 그 결과가 배열이 아니면 빈 배열 `[]`을 사용한다. 이를 통해 단일 자식(`child`)과 복수 자식(`children`) 속성을 모두 처리한다.
3. `node.weight`가 정의된 경우 `{'--weight': node.weight}` CSS 변수 객체를 `hostStyle`로 생성한다.

**자식 렌더링 로직:**

`children` 배열을 map하여 각 항목마다 다음을 수행한다.

- `childId` 결정: 항목이 객체이고 `'id'` 속성이 있으면 `(child as Types.AnyComponentNode).id`를 사용하고, 없으면 `child-${index}`를 fallback으로 사용한다.
- `childNode` 결정: 항목이 객체이고 `'type'` 속성이 있으면 `Types.AnyComponentNode`로 캐스팅하고, 아니면 `null`을 전달한다.
- `<ComponentNode key={childId} node={childNode} surfaceId={surfaceId} />`를 렌더링한다.

**렌더링 구조:**

최상위 `<div className="a2ui-card">`에 `hostStyle`을 적용한다. 내부 `<section>`에 `classMapToString(theme.components.Card)` 클래스와 `stylesToObject(theme.additionalStyles?.Card)` 스타일을 적용하고, 자식 `ComponentNode`들을 렌더링한다.

## 동작 흐름

props에서 단일 또는 복수 자식 노드를 추출하여 배열로 정규화한 뒤, 각 자식을 `ComponentNode`에 위임하여 재귀적으로 렌더링한다. 레이아웃과 스타일은 전적으로 theme 클래스에 의존한다.
