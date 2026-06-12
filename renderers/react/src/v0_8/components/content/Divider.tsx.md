# renderers/react/src/v0_8/components/content/Divider.tsx

## 개요

`Divider` 컴포넌트는 시각적 구분선(`<hr>`)을 렌더링하는 단순한 컴포넌트다. A2UI 노드 트리의 `DividerNode`를 받아 테마 클래스가 적용된 `<hr>` 요소를 출력하며, flex 레이아웃에서 `--weight` CSS 변수를 루트 div에 적용한다. Lit 렌더러의 `Divider` 컴포넌트 구조를 그대로 따른다.

## 의존성

### 외부 패키지
- `react` — `memo`
- `@a2ui/web_core/types/types` — `Types.DividerNode` (타입 전용)

### 저장소 내부 모듈
- [`../../types`](../../types.ts.md) — `A2UIComponentProps` 타입
- [`../../hooks/useA2UIComponent`](../../hooks/useA2UIComponent.ts.md) — `useA2UIComponent` 훅
- [`../../lib/utils`](../../lib/utils.ts.md) — `classMapToString`, `stylesToObject`

## Exports

- `Divider` — named export, memo로 감싼 함수형 컴포넌트
- `default` — `Divider`의 default export

## 상세 명세

### `Divider`

**시그니처**: `memo(function Divider({ node, surfaceId }: A2UIComponentProps<Types.DividerNode>): JSX.Element)`

**동작 단계:**
1. `useA2UIComponent(node, surfaceId)`를 호출하여 `theme`을 획득한다 (문자열 해석 불필요).
2. `node.weight`가 `undefined`가 아니면 `{'--weight': node.weight}`를 `hostStyle`에 설정하고, 그렇지 않으면 `{}`를 사용한다.
3. 루트 `<div className="a2ui-divider" style={hostStyle}>`를 렌더링한다.
4. 내부에 `<hr className={classMapToString(theme.components.Divider)} style={stylesToObject(theme.additionalStyles?.Divider)} />`를 렌더링한다.

**경계 케이스:**
- 조기 반환 조건 없음. 노드가 전달되면 항상 구분선을 렌더링한다.
- `node.weight`가 없으면 `--weight` CSS 변수를 설정하지 않는다.

## 동작 흐름

테마 획득 → 호스트 스타일 계산 → `:host` 역할의 루트 div + 테마 클래스 적용된 `<hr>` 렌더링.
