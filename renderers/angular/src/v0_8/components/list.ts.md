# renderers/angular/src/v0_8/components/list.ts

## 개요

`List`는 자식 컴포넌트들을 스크롤 가능한 목록으로 렌더링하는 Angular 컴포넌트다. `direction` 입력에 따라 세로(`vertical`) 또는 가로(`horizontal`) 스크롤 레이아웃을 적용하며, 각 자식은 `.a2ui-list-item` div로 감싸진다. 호스트 요소의 `direction` 속성에 값을 바인딩하여 CSS 속성 선택자(`[direction='vertical']`)로 레이아웃을 전환한다.

## 의존성

### 외부 패키지
- `@angular/core`: `ChangeDetectionStrategy`, `Component`, `input`

### 저장소 내부 모듈
- [`../types`](../types.ts.md): `AnyComponentNode`, `ListNode`, `ResolvedList` 타입
- [`../rendering/dynamic-component`](../rendering/dynamic-component.ts.md): `DynamicComponent` 기반 클래스
- [`../rendering/renderer`](../rendering/renderer.ts.md): `Renderer` 디렉티브

## Exports

| 이름 | 종류 |
|------|------|
| `List` | 클래스 (Angular 컴포넌트) |

## 상세 명세

### `List` 클래스

**선언:** `export class List extends DynamicComponent<ListNode>`

**데코레이터 설정:**
- `selector`: `'a2ui-list'`
- `imports`: `[Renderer]`
- `changeDetection`: `ChangeDetectionStrategy.OnPush`
- `host`: `{'[attr.direction]': 'direction()'}` — 호스트 요소의 `direction` 속성에 현재 방향 값을 바인딩

**인라인 스타일:**
- `:host`: `display: block; flex: var(--weight); min-height: 0;`
- `:host([direction='vertical']) section`: `display: flex; flex-direction: column; max-height: 100%; overflow-y: auto;`
- `:host([direction='horizontal']) section`: `display: flex; max-width: 100%; overflow-x: auto; overflow-y: hidden; scrollbar-width: none;`
- `.a2ui-list-item`: `display: flex; cursor: pointer; box-sizing: border-box;`

**템플릿 동작:**
- `<section [class]="theme.components.List" [style]="theme.additionalStyles?.List">`
- `children()` 또는 `component().properties.children` 배열을 `@for`로 순회
- 각 `child`가 truthy이면 `<div class="a2ui-list-item">` 로 감싸서 `<ng-container a2ui-renderer>` 로 렌더링
- `track`은 `child?.id ?? child` 사용

**입력 신호(Inputs):**

| 입력 이름 | 타입 | 기본값 |
|-----------|------|--------|
| `alignment` | `ResolvedList['alignment']` | `'stretch'` |
| `direction` | `ResolvedList['direction']` | `'vertical'` |
| `children` | `AnyComponentNode[] \| null` | `null` |

## 동작 흐름

1. `direction()` 신호 값이 호스트의 `direction` 속성에 반영되어 CSS 선택자가 레이아웃 모드를 결정한다.
2. `children()` 입력이 있으면 그것을, 없으면 `component().properties.children`을 목록 소스로 사용한다.
3. 유효한 자식마다 `.a2ui-list-item` 컨테이너 안에 `a2ui-renderer`로 렌더링한다.
