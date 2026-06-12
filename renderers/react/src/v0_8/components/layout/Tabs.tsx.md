# renderers/react/src/v0_8/components/layout/Tabs.tsx

## 개요

`Tabs` 컴포넌트는 여러 탭 항목 중 하나를 선택하여 해당 탭의 콘텐츠를 표시하는 레이아웃 컨테이너다. 탭 버튼 영역과 콘텐츠 영역으로 구성되며, 현재 선택된 탭은 theme의 selected 클래스를 all 클래스와 병합하고 비활성화 처리된다. Lit 렌더러의 `Tabs` 커스텀 엘리먼트와 동일한 구조를 유지한다.

## 의존성

### 외부 패키지
- `react`: `useState`, `memo`
- `@a2ui/web_core/types/types`: `Types.TabsNode` (타입 전용 import)

### 저장소 내부 모듈
- [`../../types`](../../types.ts.md): `A2UIComponentProps` 타입
- [`../../hooks/useA2UIComponent`](../../hooks/useA2UIComponent.ts.md): `useA2UIComponent` 훅
- [`../../lib/utils`](../../lib/utils.ts.md): `classMapToString`, `stylesToObject`, `mergeClassMaps` 유틸
- [`../../core/ComponentNode`](../../core/ComponentNode.tsx.md): `ComponentNode` 재귀 렌더러

## Exports

- `Tabs` — memo로 감싼 named export (React 함수 컴포넌트)
- `default` — `Tabs`를 default export로도 제공

## 상세 명세

### `Tabs` (컴포넌트)

**시그니처:** `memo(function Tabs({ node, surfaceId }: A2UIComponentProps<Types.TabsNode>): JSX.Element)`

**상태:**

- `selectedIndex: number` — 현재 선택된 탭의 인덱스. 초기값 `0`.

**내부 초기화:**

1. `useA2UIComponent(node, surfaceId)`로 `theme`, `resolveString`을 가져온다.
2. `useState(0)`으로 `selectedIndex` 상태를 초기화한다 (기본값: 첫 번째 탭 선택).
3. `props.tabItems ?? []`로 탭 항목 배열을 가져온다.
4. `node.weight`가 정의된 경우 `{'--weight': node.weight}` CSS 변수 객체를 `hostStyle`로 생성한다.

**탭 버튼 렌더링:**

`tabItems.map((tab, index) => ...)` 순회. 각 탭마다:

- `title`은 `resolveString(tab.title)`로 결정한다.
- `isSelected`는 `index === selectedIndex`로 결정한다.
- 클래스 결정: `isSelected`이면 `mergeClassMaps(theme.components.Tabs.controls.all, theme.components.Tabs.controls.selected)`로 all 클래스와 selected 클래스를 병합하고, 아니면 `theme.components.Tabs.controls.all`만 사용한다. Lit이 선택 시 두 클래스셋을 모두 적용하는 동작을 그대로 재현한다.
- `<button key={index} disabled={isSelected} className={classMapToString(classes)} onClick={() => setSelectedIndex(index)}>`: 선택된 탭은 `disabled`로 재클릭을 방지한다.

**탭 콘텐츠 렌더링:**

`tabItems[selectedIndex]`가 존재하면 `<ComponentNode node={tabItems[selectedIndex].child} surfaceId={surfaceId} />`를 렌더링한다. 탭 항목이 없거나 `selectedIndex`가 범위를 벗어난 경우 아무것도 렌더링하지 않는다.

**렌더링 구조:**

최상위 `<div className="a2ui-tabs">`에 `hostStyle`을 적용한다. 내부 `<section>`에 `classMapToString(theme.components.Tabs.container)` 클래스와 `stylesToObject(theme.additionalStyles?.Tabs)` 스타일을 적용한다. `<section>` 안에는 탭 버튼들을 담은 `<div id="buttons" className={classMapToString(theme.components.Tabs.element)}>`와 선택된 탭의 `ComponentNode`가 순서대로 배치된다.

## 동작 흐름

초기에는 인덱스 0의 탭이 선택된다. 사용자가 다른 탭 버튼을 클릭하면 `setSelectedIndex`가 호출되어 `selectedIndex`가 변경되고, 해당 인덱스의 탭 콘텐츠가 `ComponentNode`를 통해 렌더링된다. 이전 콘텐츠는 DOM에서 제거되고 새 콘텐츠가 마운트된다(조건부 렌더링). 선택된 탭 버튼은 `disabled` 처리와 selected 클래스 병합을 통해 시각적으로 구분된다.
