# renderers/angular/src/v0_9/catalog/basic/column.component.ts

## 개요

A2UI v0.9 Column 컴포넌트의 Angular 구현체다. 자식 컴포넌트들을 수직 flex 레이아웃으로 배치하며, 정적 자식 목록과 데이터 컬렉션에 바인딩된 반복 템플릿을 모두 지원한다. 호스트 엘리먼트에 인라인 스타일을 직접 설정해 `display: flex`, `flex-direction: column`, `width: 100%` 레이아웃을 강제하고, `justify-content`와 `align-items`를 computed 시그널로 반응형 제어한다.

## 의존성

### 외부 패키지
- `@angular/core` — `Component`, `computed`, `ChangeDetectionStrategy`
- `@a2ui/web_core/v0_9/basic_catalog` — `ColumnApi`

### 저장소 내부 모듈
- [`../../core/component-host.component`](../../core/component-host.component.ts.md) — `ComponentHostComponent`
- [`../../core/component-binder.service`](../../core/component-binder.service.ts.md) — `Child`
- [`./basic-catalog-component`](./basic-catalog-component.ts.md) — `BasicCatalogComponent`
- [`./utils`](./utils.ts.md) — `JUSTIFY_MAP`, `ALIGN_MAP`

## Exports

- `ColumnComponent` — Angular standalone 컴포넌트 클래스

## 상세 명세

### `ColumnComponent`

`BasicCatalogComponent<typeof ColumnApi>`를 상속하는 Angular standalone 컴포넌트.

**데코레이터 설정:**
- `selector`: `'a2ui-v09-column'`
- `standalone: true`
- `imports`: `[ComponentHostComponent]`
- `changeDetection`: `ChangeDetectionStrategy.OnPush`
- `host` 바인딩:
  - `[style.display]`: 항상 리터럴 `"flex"`
  - `[style.flex-direction]`: 항상 리터럴 `"column"`
  - `[style.width]`: 항상 리터럴 `"100%"`
  - `[style.gap]`: 항상 리터럴 `"var(--a2ui-column-gap, var(--a2ui-spacing-m, 16px))"` (CSS 변수, 최종 기본값 16px)
  - `[style.justify-content]`: `justify()` 시그널 반환값
  - `[style.align-items]`: `align()` 시그널 반환값

**computed 시그널:**

- `justify` (`protected readonly`, 반환 타입 `string | undefined`):
  `props()['justify']?.value()`를 읽는다. 값이 존재하면 `JUSTIFY_MAP[val]`을 먼저 조회하고 매핑이 있으면 그 값을 반환한다. 매핑이 없으면 `val` 원래 값을 반환한다. 값 자체가 falsy이면 `undefined`를 반환한다.

- `align` (`protected readonly`, 반환 타입 `string | undefined`):
  `props()['align']?.value()`를 읽는다. 값이 존재하면 `ALIGN_MAP[val]`을 먼저 조회하고 매핑이 있으면 그 값을 반환한다. 매핑이 없으면 `val` 원래 값을 반환한다. 값 자체가 falsy이면 `undefined`를 반환한다.

- `children` (`protected readonly`, 반환 타입 `Child[]`):
  `props()['children'].value()` 결과를 반환한다. 결과가 falsy(null/undefined)이면 빈 배열 `[]`을 반환한다. (`?.` 없이 직접 접근 — `children`은 필수 프롭으로 간주.)

**메서드:**

- `trackChild(_index: number, child: Child): string`
  - 접근 제어자: `protected`
  - `@for` 루프의 `track` 함수로 사용된다.
  - `` `${child.basePath}/${child.id}` `` 형태의 복합 키 문자열을 반환한다.
  - Angular 변경 감지가 동일한 자식 항목을 안정적으로 재사용할 수 있도록 한다.

**템플릿 동작:**

`@for (child of children(); track trackChild($index, child))` 루프로 `children()` 배열을 순회하며, 각 항목에 대해 `<a2ui-v09-component-host [componentKey]="child" [surfaceId]="surfaceId()">` 엘리먼트를 렌더링한다.

## 동작 흐름

1. `BasicCatalogComponent`로부터 `props()` 시그널을 상속받는다.
2. `justify`, `align`, `children`을 computed 시그널로 파생한다.
3. 호스트 엘리먼트에 `display: flex`, `flex-direction: column`, `width: 100%`, `gap` 스타일을 정적으로 적용한다.
4. `justify-content`, `align-items`는 각 computed 시그널 값에 반응해 업데이트된다.
5. 템플릿에서 `children()` 배열을 순회하며 각 자식을 `ComponentHostComponent`로 위임 렌더링한다.
6. `trackChild` 함수로 리스트 항목의 동일성을 `basePath/id` 복합 키로 추적한다.
