# renderers/angular/src/v0_9/catalog/basic/tabs.component.ts

## 개요

A2UI v0.9 Tabs 컴포넌트의 Angular 구현체다. 탭 버튼 목록을 상단에 렌더링하고, 활성 탭에 해당하는 자식 컴포넌트 콘텐츠를 하단에 표시한다. 활성 탭 인덱스를 내부 signal로 관리하며, 자식 컴포넌트는 `ComponentHostComponent`를 통해 동적으로 렌더링된다.

## 의존성

### 외부 패키지
- `@angular/core`: `Component`, `computed`, `ChangeDetectionStrategy`, `signal`
- `@a2ui/web_core/v0_9/basic_catalog`: `TabsApi`

### 저장소 내부 모듈
- [`../../core/component-host.component`](../../core/component-host.component.ts.md): 활성 탭 콘텐츠 렌더링
- [`./basic-catalog-component`](./basic-catalog-component.ts.md): 기반 추상 클래스

## Exports

- `TabsComponent` — Angular standalone 컴포넌트 클래스

## 상세 명세

### `TabsComponent`

**데코레이터**: `@Component`
- `selector`: `'a2ui-v09-tabs'`
- `standalone: true`
- `imports`: `[ComponentHostComponent]`
- `changeDetection`: `ChangeDetectionStrategy.OnPush`

**상속**: `BasicCatalogComponent<typeof TabsApi>`

**필드 및 computed 속성**

| 이름 | 종류 | 타입 | 설명 |
|------|------|------|------|
| `activeTabIndex` | `WritableSignal` | `number` | 현재 활성 탭 인덱스. 초기값 `0` |
| `tabs` | `computed Signal` | `Array<{title: DynamicString; child: string}>` | `props()['tabs']?.value() \|\| []`. tabs 배열. 없으면 빈 배열 |
| `activeTab` | `computed Signal` | 단일 탭 객체 or undefined | `tabs()[activeTabIndex()]`. 현재 활성 탭 |
| `normalizedActiveTabChild` | `computed Signal` (protected) | `{id: string; basePath: string} \| null` | 활성 탭의 child를 정규화 |

#### `normalizedActiveTabChild` 정규화 로직

1. `activeTab()?.child`를 읽는다. 값이 없으면 `null` 반환.
2. `child`가 `object` 타입이고 `'id'` 키를 가지면 그대로 `{id, basePath}` 형태로 캐스팅하여 반환한다.
3. `child`가 `string`이면 `{id: child, basePath: this.dataContextPath()}`로 변환하여 반환한다.

**메서드**

#### `setActiveTab(index: number): void`
`activeTabIndex.set(index)`를 호출하여 활성 탭을 변경한다.

**템플릿 구조**

1. `div.a2ui-tabs` 안에 `div.a2ui-tab-bar`와 조건부 `div.a2ui-tab-content`가 세로로 배치된다.
2. `a2ui-tab-bar`에서 `tabs()` 배열을 `@for`로 순회하며 각 탭에 `button.a2ui-tab-button`을 렌더링한다. 활성 탭이면 `.active` 클래스가 추가된다. 클릭 시 `setActiveTab(i)` 호출.
3. `normalizedActiveTabChild()`가 null이 아닐 때만 `a2ui-tab-content`를 렌더링하고, 내부에 `<a2ui-v09-component-host>`를 배치한다.

**스타일 (CSS 변수 지원)**

| CSS 변수 | 기본값 | 역할 |
|----------|--------|------|
| `--a2ui-tabs-border` | `2px solid var(--a2ui-color-border, #eee)` | 탭 바 하단 테두리 |
| `--a2ui-tabs-header-background` | `transparent` | 비활성 탭 버튼 배경 |
| `--a2ui-tabs-header-color` | `var(--a2ui-text-caption-color, #666)` | 비활성 탭 버튼 텍스트 색상 |
| `--a2ui-tabs-header-background-active` | `transparent` | 활성 탭 버튼 배경 |
| `--a2ui-tabs-header-color-active` | `var(--a2ui-color-primary, #007bff)` | 활성 탭 버튼 텍스트 색상 |
| `--a2ui-tabs-content-padding` | `var(--a2ui-spacing-m, 16px) 0` | 탭 콘텐츠 패딩 |

활성 탭 버튼은 하단에 `2px solid var(--a2ui-color-primary, #007bff)` 밑줄이 추가되며, `margin-bottom: -2px`로 탭 바 테두리와 겹쳐 자연스럽게 연결된다.

## 동작 흐름

컴포넌트가 초기화되면 `activeTabIndex`는 `0`으로 시작한다. `tabs()` 배열이 변경되어도 인덱스는 리셋되지 않으므로, 탭 목록이 줄어드는 경우에는 `activeTab()`이 `undefined`가 될 수 있다. 사용자가 탭 버튼을 클릭하면 `setActiveTab(i)`이 인덱스를 갱신하고, `activeTab` → `normalizedActiveTabChild` 계산이 연쇄적으로 실행되어 `ComponentHostComponent`에 새로운 `componentKey`가 전달된다. child 값의 형태가 문자열이든 객체든 `normalizedActiveTabChild`가 항상 `{id, basePath}` 객체로 정규화한다.
