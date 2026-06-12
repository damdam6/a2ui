# renderers/angular/src/v0_8/components/tabs.ts

## 개요

`Tabs` 컴포넌트는 탭 컨트롤 버튼과 선택된 탭의 자식 컴포넌트를 렌더링하는 Angular 독립형 컴포넌트다. `DynamicComponent<TabsNode>`를 상속하며, `signal`로 현재 선택된 탭 인덱스를 관리한다. 한 번에 하나의 탭 컨텐츠만 렌더링하며 탭 버튼 클릭으로 전환된다.

## 의존성

### 외부 패키지
- `@angular/core`: `ChangeDetectionStrategy`, `Component`, `input`, `signal`

### 저장소 내부 모듈
- [`../rendering/dynamic-component`](../rendering/dynamic-component.ts.md) — `DynamicComponent` 기반 클래스
- [`../rendering/renderer`](../rendering/renderer.ts.md) — `Renderer` 디렉티브
- [`../types`](../types.ts.md) — `ResolvedTabs`, `TabsNode` 타입

## Exports

| 이름 | 종류 |
|------|------|
| `Tabs` | 클래스 (Angular Component) |

## 상세 명세

### `Tabs` 클래스

`@Component` 데코레이터 설정:
- `selector`: `'a2ui-tabs'`
- `imports`: `[Renderer]`
- `changeDetection`: `ChangeDetectionStrategy.OnPush`
- `styles`: `:host { display: block; }`, `.a2ui-tabs-content { flex: 1; min-height: 0; }`

`DynamicComponent<TabsNode>`를 상속한다.

#### 필드

| 이름 | 타입 | 기본값 | 설명 |
|------|------|--------|------|
| `tabItems` | `InputSignal<ResolvedTabs['tabItems']>` | required | 각 탭의 `title`(StringValue)과 `child`(컴포넌트 노드) 배열 |
| `selectedIndex` | `WritableSignal<number>` (protected) | `signal(0)` | 현재 선택된 탭의 인덱스, 초기값 0 |

#### `selectTab(index: number): void` (protected)
`selectedIndex` 시그널을 전달받은 `index`로 설정한다. 탭 버튼의 `(click)` 이벤트 핸들러로 바인딩된다.

#### 템플릿 구조
최상위 div에 `theme.components.Tabs.container` 클래스와 `theme.additionalStyles?.Tabs` 스타일 적용.

1. 컨트롤 div: `theme.components.Tabs.controls.all` 클래스.
   - `@for (item of tabItems(); track item.child; let i = $index)` 루프로 `<button>` 렌더링.
   - 각 버튼: `[class]="selectedIndex() === i ? theme.components.Tabs.controls.selected : {}"` 바인딩으로 선택된 탭에만 selected 클래스 적용. `(click)="selectTab(i)"`. 텍스트는 `resolvePrimitive(item.title)`.
2. 컨텐츠 div: `class="a2ui-tabs-content"`.
   - `@if (tabItems()[selectedIndex()]; as selectedTab)`: 현재 선택된 탭 항목이 있으면 `<ng-container a2ui-renderer [surfaceId]="surfaceId()!" [component]="selectedTab.child" />`를 렌더링.

## 동작 흐름

1. 초기 렌더링 시 `selectedIndex`는 0이므로 첫 번째 탭이 선택된 상태로 표시된다.
2. 모든 탭의 제목 버튼이 `@for`로 렌더링된다.
3. 선택된 탭의 버튼에만 `theme.components.Tabs.controls.selected` 클래스가 적용된다.
4. 컨텐츠 영역에는 `tabItems()[selectedIndex()]`의 `child` 노드만 렌더링된다 (단일 렌더).
5. 사용자가 탭 버튼 클릭 → `selectTab(i)` → `selectedIndex` 변경 → OnPush 변경 감지 트리거 → 선택 클래스와 컨텐츠가 즉시 교체된다.
