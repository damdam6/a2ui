# renderers/angular/src/v0_9/catalog/basic/row.component.ts

## 개요

A2UI v0.9 Row 컴포넌트의 Angular 구현체다. 자식 컴포넌트들을 가로 방향 flex 레이아웃으로 배치하며, 정적 자식 목록과 데이터 컬렉션에 바인딩된 반복 템플릿 자식 모두를 지원한다. `justify` 및 `align` 프로퍼티를 A2UI 의미값에서 CSS 값으로 변환하여 호스트 엘리먼트의 인라인 스타일에 바인딩한다.

## 의존성

### 외부 패키지
- `@angular/core`: `Component`, `computed`, `ChangeDetectionStrategy`
- `@a2ui/web_core/v0_9/basic_catalog`: `RowApi`

### 저장소 내부 모듈
- [`../../core/component-host.component`](../../core/component-host.component.ts.md): 자식 컴포넌트 동적 렌더링
- [`../../core/component-binder.service`](../../core/component-binder.service.ts.md): `Child` 인터페이스 타입 참조
- [`./basic-catalog-component`](./basic-catalog-component.ts.md): 기반 추상 클래스
- [`./utils`](./utils.ts.md): `JUSTIFY_MAP`, `ALIGN_MAP` 상수

## Exports

- `RowComponent` — Angular standalone 컴포넌트 클래스

## 상세 명세

### `RowComponent`

**데코레이터**: `@Component`
- `selector`: `'a2ui-v09-row'`
- `standalone: true`
- `imports`: `[ComponentHostComponent]`
- `changeDetection`: `ChangeDetectionStrategy.OnPush`

**상속**: `BasicCatalogComponent<typeof RowApi>`

**호스트 바인딩** (컴포넌트 호스트 엘리먼트에 직접 적용)

| 바인딩 | 값 |
|-------|----|
| `display` | 항상 `"flex"` |
| `flex-direction` | 항상 `"row"` |
| `gap` | `"var(--a2ui-row-gap, var(--a2ui-spacing-m, 16px))"` |
| `justify-content` | `justify()` computed 결과 |
| `align-items` | `align()` computed 결과 |

**computed 필드**

#### `justify` (protected): `Signal<string | undefined>`
`props()['justify']?.value()`를 읽어 `JUSTIFY_MAP`으로 CSS 값으로 변환한다. 값이 없으면 `undefined`를 반환하고, 맵에 없는 값이면 원래 값을 그대로 반환한다.

#### `align` (protected): `Signal<string | undefined>`
`props()['align']?.value()`를 읽어 `ALIGN_MAP`으로 CSS 값으로 변환한다. 로직은 `justify`와 동일하다.

#### `children` (protected): `Signal<Child[]>`
`props()['children'].value() || []`를 반환한다. `Child`는 `{id: string, basePath: string}` 형태다.

**메서드**

#### `trackChild(_index: number, child: Child): string` (protected)
`@for` 루프의 `track` 함수. `"${child.basePath}/${child.id}"` 형식의 고유 문자열을 반환하여 Angular의 DOM 재사용을 최적화한다.

**템플릿**

`children()` 배열을 `@for`로 순회하며, 각 항목마다 `<a2ui-v09-component-host>`를 렌더링한다. `[componentKey]`에 `child` 객체를, `[surfaceId]`에 `surfaceId()`를 바인딩한다.

## 동작 흐름

컴포넌트가 초기화되면 `props()`에서 `justify`, `align`, `children` 값을 읽어 computed signal로 노출한다. `justify`와 `align`은 호스트 엘리먼트의 인라인 스타일로 직접 적용되어 별도 CSS 래퍼 없이도 레이아웃이 동작한다. `children` 배열이 변경되면 `@for` 루프가 `trackChild` 기준으로 최소한의 DOM 업데이트를 수행한다. 반복 템플릿(`basePath`가 다른 동일 `id`)도 동일한 방식으로 처리되어 데이터 기반 리스트 렌더링을 지원한다.
