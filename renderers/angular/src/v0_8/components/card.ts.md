# renderers/angular/src/v0_8/components/card.ts

## 개요

a2ui v0.8 명세의 `Card` UI 컴포넌트를 구현하는 Angular 스탠드얼론 컴포넌트 파일이다. `DynamicComponent`를 상속하며, 단일 자식(`child`) 또는 복수 자식(`children`) 컴포넌트 노드를 카드 컨테이너 내부에서 동적으로 렌더링한다. 테마의 `components.Card`와 `additionalStyles.Card`를 래퍼 `<div>`에 적용한다.

## 의존성

### 외부 패키지
- `@angular/core` — `ChangeDetectionStrategy`, `Component`, `input`

### 저장소 내부 모듈
- [`../rendering/dynamic-component`](../rendering/dynamic-component.ts.md) — `DynamicComponent` 기반 클래스
- [`../rendering/renderer`](../rendering/renderer.ts.md) — `Renderer` 디렉티브
- `../types` (타입 전용) — `AnyComponentNode`, `CardNode`

## Exports

| 이름 | 종류 |
|------|------|
| `Card` | 클래스 (Angular 컴포넌트) |

## 상세 명세

### 클래스: `Card`
- 데코레이터: `@Component`
- 셀렉터: `a2ui-card`
- `imports`: `[Renderer]`
- `changeDetection`: `ChangeDetectionStrategy.OnPush`
- 상속: `DynamicComponent<CardNode>`

#### 템플릿
최상위 `<div>` 요소에 `[class]="theme.components.Card"`와 `[style]="theme.additionalStyles?.Card"`를 바인딩한다. 내부에 두 가지 렌더링 경로가 있다.

첫 번째로, `@if (child())` 블록에서 `child()`가 truthy이면 단일 자식 노드를 `a2ui-renderer`로 렌더링한다. `[component]="child()!"` 바인딩에서 non-null assertion을 사용한다.

두 번째로, `@for (comp of children(); track comp.id)` 블록에서 `children()` 배열을 순회하며 각 컴포넌트 노드를 개별 `a2ui-renderer`로 렌더링한다. `track comp.id`로 최소 DOM 갱신을 보장한다.

#### 스타일
- `:host` — `display: block`

#### 입력 (Signal input)

| 이름 | 타입 | 기본값 |
|------|------|--------|
| `child` | `AnyComponentNode \| null` | `null` |
| `children` | `AnyComponentNode[]` | `[]` |

## 동작 흐름

컴포넌트 초기화 시 `child`와 `children` 신호가 설정된다. `child`가 있으면 단일 자식을 렌더링하고, `children`이 있으면 배열을 순회하며 각 노드를 렌더링한다. 두 경로는 독립적으로 작동하므로 동시에 활성화될 수 있다. 입력 변경 시 OnPush 변경 감지에 의해 DOM이 갱신된다.
