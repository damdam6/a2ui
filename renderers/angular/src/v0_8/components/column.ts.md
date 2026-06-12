# renderers/angular/src/v0_8/components/column.ts

## 개요

`Column`은 자식 컴포넌트들을 세로 방향으로 배치하는 레이아웃 컨테이너 Angular 컴포넌트다. Flexbox 기반의 `<section>` 요소를 렌더링하며, 정렬(alignment)과 분배(distribution) 방식을 CSS 클래스로 제어한다. `DynamicComponent`를 상속하여 A2UI 렌더러 파이프라인에 통합되며, 자식 목록을 외부 입력 또는 컴포넌트 노드의 `properties.children`에서 가져올 수 있다.

## 의존성

### 외부 패키지
- `@angular/core`: `ChangeDetectionStrategy`, `Component`, `computed`, `input`

### 저장소 내부 모듈
- [`../types`](../types.ts.md): `AnyComponentNode`, `ColumnNode`, `ResolvedColumn` 타입
- [`../rendering/dynamic-component`](../rendering/dynamic-component.ts.md): `DynamicComponent` 기반 클래스
- [`../rendering/renderer`](../rendering/renderer.ts.md): `Renderer` 디렉티브 (자식 렌더링에 사용)

## Exports

| 이름 | 종류 |
|------|------|
| `Column` | 클래스 (Angular 컴포넌트) |

## 상세 명세

### `Column` 클래스

**선언:** `export class Column extends DynamicComponent<ColumnNode>`

**데코레이터 설정:**
- `selector`: `'a2ui-column'`
- `imports`: `[Renderer]`
- `changeDetection`: `ChangeDetectionStrategy.OnPush`

**인라인 스타일:**
- `:host`는 `display: flex`, `flex: var(--weight)` — 부모 flex 컨테이너 내에서 가중치 기반으로 크기 결정
- `section`은 `display: flex; flex-direction: column; min-width: 100%; height: 100%; box-sizing: border-box`
- `.align-{start|center|end|stretch}` 클래스 각각 `align-items` 값 설정
- `.distribute-{start|center|end|spaceBetween|spaceAround|spaceEvenly}` 클래스 각각 `justify-content` 값 설정

**템플릿 동작:**
- `<section>` 에 `[class]="classes()"`, `[style]="theme.additionalStyles?.Column"` 바인딩
- `children()` 또는 `component().properties.children` 배열을 `@for`로 순회하며, `child`가 truthy일 때만 `<ng-container a2ui-renderer>`로 각 자식 컴포넌트를 렌더링
- `track`은 `child?.id ?? child` 사용

**입력 신호(Inputs):**

| 입력 이름 | 타입 | 기본값 |
|-----------|------|--------|
| `alignment` | `ResolvedColumn['alignment']` | `'stretch'` |
| `distribution` | `ResolvedColumn['distribution']` | `'start'` |
| `children` | `AnyComponentNode[] \| null` | `undefined` |

**computed 필드:**

- `classes` — 보호된(`protected`) computed 신호. `this.theme.components.Column`의 기본 클래스 객체에 `align-${alignment()}` 키와 `distribute-${distribution()}` 키를 `true`로 추가해 반환한다. 스프레드 연산자를 사용해 베이스 클래스 맵과 동적 클래스를 병합한다.

## 동작 흐름

1. 부모가 `alignment`, `distribution`, `children` 입력을 전달한다.
2. `classes()` computed 신호가 테마의 기본 Column 클래스와 정렬/분배 클래스를 병합한다.
3. `<section>` 이 동적 클래스와 테마 추가 스타일을 적용받는다.
4. `children()` 값이 있으면 그것을, 없으면 `component().properties.children`을 순회하며 각 자식을 `a2ui-renderer`로 렌더링한다.
