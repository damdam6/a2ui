# renderers/angular/src/v0_8/components/row.ts

## 개요

`Row` 컴포넌트는 자식 컴포넌트 노드들을 가로 방향(flex-direction: row)으로 배열하는 레이아웃 컴포넌트다. `DynamicComponent<RowNode>`를 상속하며 `alignment`와 `distribution` 입력값에 따라 CSS 클래스를 동적으로 적용한다. 자식 노드 목록은 외부 입력 또는 컴포넌트 노드의 `properties.children`에서 가져온다.

## 의존성

### 외부 패키지
- `@angular/core`: `ChangeDetectionStrategy`, `Component`, `computed`, `input`

### 저장소 내부 모듈
- [`../rendering/dynamic-component`](../rendering/dynamic-component.ts.md) — `DynamicComponent` 기반 클래스
- [`../rendering/renderer`](../rendering/renderer.ts.md) — `Renderer` 디렉티브
- [`../types`](../types.ts.md) — `AnyComponentNode`, `ResolvedRow`, `RowNode` 타입

## Exports

| 이름 | 종류 |
|------|------|
| `Row` | 클래스 (Angular Component) |

## 상세 명세

### `Row` 클래스

`@Component` 데코레이터 설정:
- `selector`: `'a2ui-row'`
- `imports`: `[Renderer]`
- `changeDetection`: `ChangeDetectionStrategy.OnPush`
- `host`: `[attr.alignment]`를 `alignment()`, `[attr.distribution]`을 `distribution()`에 바인딩하여 호스트 요소에 HTML 어트리뷰트로 노출.

`DynamicComponent<RowNode>`를 상속한다.

#### 필드

| 이름 | 타입 | 기본값 | 설명 |
|------|------|--------|------|
| `alignment` | `InputSignal<ResolvedRow['alignment']>` | `'stretch'` | 교차축 정렬. 가능한 값: `'start'`, `'center'`, `'end'`, `'stretch'` |
| `distribution` | `InputSignal<ResolvedRow['distribution']>` | `'start'` | 주축 분배. 가능한 값: `'start'`, `'center'`, `'end'`, `'spaceBetween'`, `'spaceAround'`, `'spaceEvenly'` |
| `children` | `InputSignal<AnyComponentNode[] \| null \| undefined>` | 선택적 | 외부에서 명시적으로 전달되는 자식 노드 목록 |

#### `classes` computed (protected)
`theme.components.Row` 객체를 전개(spread)하고 `align-${alignment()}` 키를 `true`, `distribute-${distribution()}` 키를 `true`로 추가한 객체를 반환한다. Angular의 `[class]` 바인딩에서 객체 형태를 사용하므로 truthy 키에 해당하는 클래스가 적용된다.

#### CSS 클래스 매핑 (컴포넌트 내부 스타일)

alignment 값 → CSS:
- `start` → `align-items: start`
- `center` → `align-items: center`
- `end` → `align-items: end`
- `stretch` → `align-items: stretch`

distribution 값 → CSS:
- `start` → `justify-content: start`
- `center` → `justify-content: center`
- `end` → `justify-content: end`
- `spaceBetween` → `justify-content: space-between`
- `spaceAround` → `justify-content: space-around`
- `spaceEvenly` → `justify-content: space-evenly`

`:host` 스타일: `display: flex`, `flex: var(--weight)`. `section`: `display: flex`, `flex-direction: row`, `width: 100%`, `min-height: 100%`, `box-sizing: border-box`.

#### 템플릿 구조
`<section [class]="classes()" [style]="theme.additionalStyles?.Row">` 내부에 `@for (child of children() ?? component().properties.children; track child?.id ?? child)` 루프. `child`가 null이 아닐 때만 `<ng-container a2ui-renderer [surfaceId]="surfaceId()!" [component]="child" />`를 렌더링한다. `children()` 입력이 있으면 우선 사용하고, 없으면 `component().properties.children`을 폴백으로 사용한다.

## 동작 흐름

1. `alignment`와 `distribution` 입력이 변경될 때마다 `classes()` computed가 새 클래스 객체를 계산한다.
2. `children()` 또는 `component().properties.children`에서 자식 노드를 결정한다.
3. 각 자식 노드는 `Renderer` 디렉티브를 통해 동적으로 컴포넌트로 인스턴스화된다.
4. 호스트 요소에 `alignment`와 `distribution`이 HTML 어트리뷰트로 노출되어 외부 CSS 선택자에서 참조 가능하다.
