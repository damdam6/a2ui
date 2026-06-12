# renderers/react/src/v0_8/components/interactive/Button.tsx

## 개요

`Button` 컴포넌트는 클릭 이벤트를 통해 A2UI 액션을 전송하는 인터랙티브 컴포넌트다. `ButtonNode`의 `child` 속성에 지정된 자식 컴포넌트(보통 `Text` 또는 `Icon`)를 `ComponentNode`를 통해 동적으로 렌더링하며, 클릭 시 `props.action`을 서버로 디스패치한다. `useCallback`으로 클릭 핸들러를 메모이제이션하고 `memo`로 불필요한 리렌더링을 방지한다.

## 의존성

### 외부 패키지
- `react` — `useCallback`, `memo`
- `@a2ui/web_core/types/types` — `Types.ButtonNode` (타입 전용)

### 저장소 내부 모듈
- [`../../types`](../../types.ts.md) — `A2UIComponentProps` 타입
- [`../../hooks/useA2UIComponent`](../../hooks/useA2UIComponent.ts.md) — `useA2UIComponent` 훅
- [`../../lib/utils`](../../lib/utils.ts.md) — `classMapToString`, `stylesToObject`
- [`../../core/ComponentNode`](../../core/ComponentNode.tsx.md) — `ComponentNode` 컴포넌트

## Exports

- `Button` — named export, memo로 감싼 함수형 컴포넌트
- `default` — `Button`의 default export

## 상세 명세

### `Button`

**시그니처**: `memo(function Button({ node, surfaceId }: A2UIComponentProps<Types.ButtonNode>): JSX.Element)`

**동작 단계:**
1. `useA2UIComponent(node, surfaceId)`로 `theme`과 `sendAction`을 획득한다.
2. `node.properties`를 `props`에 할당한다.
3. `useCallback`으로 `handleClick` 함수를 생성한다. 의존성 배열은 `[props.action, sendAction]`이다.
   - `props.action`이 truthy이면 `sendAction(props.action)`을 호출한다.
   - `props.action`이 없으면 아무것도 하지 않는다.
4. `node.weight`가 `undefined`가 아니면 `{'--weight': node.weight}`를 `hostStyle`로 설정하고, 없으면 `{}`를 사용한다.
5. 루트 `<div className="a2ui-button" style={hostStyle}>`를 렌더링한다.
6. 내부에 `<button className={classMapToString(theme.components.Button)} style={stylesToObject(theme.additionalStyles?.Button)} onClick={handleClick}>`를 렌더링한다.
7. `<button>` 내부에 `<ComponentNode node={props.child} surfaceId={surfaceId} />`를 렌더링하여 자식 노드를 동적으로 처리한다.

**경계 케이스:**
- `props.action`이 없으면 클릭해도 아무 액션도 전송되지 않는다(핸들러는 등록되지만 조건 분기로 아무것도 하지 않음).
- `props.child`가 null/undefined인 경우 `ComponentNode` 내부에서 처리된다.
- `node.weight`가 없으면 `--weight` CSS 변수를 설정하지 않는다.

## 동작 흐름

훅으로 테마와 액션 디스패처 획득 → 클릭 핸들러 메모이제이션 → 호스트 스타일 계산 → 루트 div + `<button>` + 자식 `ComponentNode` 렌더링. 클릭 시 핸들러가 `sendAction`을 통해 액션 객체를 컨텍스트 바인딩과 함께 서버로 디스패치한다.
