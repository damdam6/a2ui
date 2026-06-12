# renderers/react/src/v0_9/catalog/basic/components/Tabs.tsx

## 개요

`Tabs` 컴포넌트는 a2ui basic catalog의 `TabsApi` 스키마를 따르는 React 구현체다. 탭 제목 목록을 헤더로 렌더링하고, 선택된 탭의 콘텐츠를 아래 영역에 표시한다. 현재 선택 인덱스는 컴포넌트 로컬 `useState`로 관리되며, 탭 타입의 TypeScript 추론 문제로 인해 내부적으로 `any` 타입 별칭을 사용한다.

## 의존성

### 외부 패키지
- `react` — `useState`, `React.CSSProperties`
- `@a2ui/web_core/v0_9/basic_catalog` — `TabsApi`

### 저장소 내부 모듈
- [`../../../adapter`](../../../adapter.tsx.md) — `createComponentImplementation`
- [`../utils`](../utils.ts.md) — `useBasicCatalogStyles`

## Exports

| 이름 | 종류 |
|------|------|
| `Tabs` | 상수 (ReactComponentImplementation) |

## 상세 명세

### `_Tab` (타입 별칭)

`type _Tab = any`

`TabsApi` 스키마 내부에 깊이 중첩된 탭 객체의 타입을 `z.infer`가 올바르게 추론하지 못하는 문제 때문에 임시로 `any`를 사용한다. 실제 탭 객체는 `{ title: string; child: ComponentId }` 형태를 가진다.

### `Tabs`

`createComponentImplementation(TabsApi, renderFn)` 호출로 생성된 컴포넌트 구현체다.

**렌더 함수 시그니처:** `({props, buildChild}) => JSX.Element`

**매개변수 (props 필드):**
- `props.tabs` — 탭 객체 배열 (옵셔널). 없으면 빈 배열 `[]`로 초기화
  - 각 탭: `{ title: string; child: ComponentId }`

**내부 상태:**
- `selectedIndex: number` — 초기값 `0`. 현재 선택된 탭의 인덱스.

**내부 파생 값:**
- `tabs`: `props.tabs || []`
- `activeTab`: `tabs[selectedIndex]`

**스타일 정의 (모두 `React.CSSProperties` 객체):**
- `tabsContainer`: `display: 'block'`
- `tabsHeaders`: `display: 'flex'`, gap은 `--a2ui-spacing-xs`(기본 `0.25rem`), `borderBottom`은 `--a2ui-tabs-border` → `--a2ui-border-width 1px solid --a2ui-color-border(#ccc)`, `marginBottom`은 `--a2ui-spacing-m`(기본 `0.5rem`)
- `tabsHeaderBase`: padding은 `--a2ui-spacing-m` × `--a2ui-spacing-l`, `background`는 `--a2ui-tabs-header-background`(기본 `transparent`), `color`는 `--a2ui-tabs-header-color` → `--a2ui-color-on-surface`, `border: 'none'`, 상단 모서리만 둥글게(`border-radius`), `cursor: 'pointer'`, `fontFamily: 'inherit'`
- `tabsHeaderActive`: `background`는 `--a2ui-tabs-header-background-active` → `--a2ui-color-secondary`(기본 `#eee`), `color`는 `--a2ui-tabs-header-color-active` → `--a2ui-color-on-secondary`(기본 `#333`)
- `content`: `padding`은 `--a2ui-tabs-content-padding` → `0 --a2ui-spacing-m(0.5rem)`

**렌더 구조:**
- `tabsContainer` 스타일의 `<div>` 최상위
- 탭 헤더 행: `tabsHeaders` 스타일의 `<div>` 안에서 `tabs.map`으로 각 탭에 대해 `<button>`을 렌더링. key는 인덱스 `i`. 클릭 시 `setSelectedIndex(i)`. 활성 탭(`selectedIndex === i`)이면 `tabsHeaderBase`에 `tabsHeaderActive`를 병합한 스타일 적용. 버튼 내용은 `tab.title`
- 콘텐츠 영역: `content` 스타일의 `<div>` 안에 `activeTab`이 존재하면 `buildChild(activeTab.child)`, 없으면 `null`

## 동작 흐름

초기 렌더링 시 `selectedIndex = 0`이므로 첫 번째 탭이 활성화된다. 탭 헤더 클릭 → `setSelectedIndex(i)` → 리렌더링 시 해당 탭 헤더에 활성 스타일이 적용되고 `activeTab`이 바뀌며 해당 탭의 자식 컴포넌트가 표시된다. `props.tabs`가 비어 있거나 `activeTab`이 없으면 콘텐츠 영역에는 아무것도 표시되지 않는다.
