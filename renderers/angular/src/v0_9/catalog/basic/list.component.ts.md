# renderers/angular/src/v0_9/catalog/basic/list.component.ts

## 개요

A2UI v0.9 List 컴포넌트의 Angular 구현체다. 자식 컴포넌트 목록을 ordered(`<ol>`), unordered(`<ul>`), 또는 무스타일(`<div>`) 방식으로 렌더링하며, 수직/수평 방향을 지원한다. `@switch` 블록으로 `listStyle` 프롭에 따라 HTML 목록 태그를 동적으로 선택하며, 각 자식은 `<a2ui-v09-component-host>`로 위임 렌더링된다.

## 의존성

### 외부 패키지
- `@angular/core` — `Component`, `computed`, `ChangeDetectionStrategy`
- `@a2ui/web_core/v0_9/basic_catalog` — `ListApi`

### 저장소 내부 모듈
- [`../../core/component-host.component`](../../core/component-host.component.ts.md) — `ComponentHostComponent`
- [`./basic-catalog-component`](./basic-catalog-component.ts.md) — `BasicCatalogComponent`
- [`../../core/component-binder.service`](../../core/component-binder.service.ts.md) — `Child`

## Exports

- `ListComponent` — Angular standalone 컴포넌트 클래스

## 상세 명세

### `ListComponent`

`BasicCatalogComponent<typeof ListApi>`를 상속하는 Angular standalone 컴포넌트.

**데코레이터 설정:**
- `selector`: `'a2ui-v09-list'`
- `standalone: true`
- `imports`: `[ComponentHostComponent]`
- `changeDetection`: `ChangeDetectionStrategy.OnPush`

**스타일 (컴포넌트 캡슐화):**

`.a2ui-list` 기본: `display: flex`, `padding-inline-start: var(--a2ui-list-padding, var(--a2ui-spacing-l, 24px))`, `margin: 0`.
- `.a2ui-list.vertical`: `flex-direction: column`, `gap: var(--a2ui-list-gap, var(--a2ui-spacing-s, 8px))`.
- `.a2ui-list.horizontal`: `flex-direction: row`, `gap: var(--a2ui-list-gap, var(--a2ui-spacing-m, 16px))`, `list-style-position: inside`.
- `.a2ui-list-item-none`: `display: block`.
- `.horizontal .a2ui-list-item-none`: `display: inline-block`.

**computed 시그널:**

- `listStyle` (`readonly`): `props()['listStyle']?.value()`. `'ordered'`, `'unordered'`, `'none'`, 또는 `undefined` 중 하나.

- `direction` (`readonly`): `props()['direction']?.value() || 'vertical'`. 기본값 `'vertical'`. `.a2ui-list` CSS 클래스에 추가된다.

- `children` (`readonly`): `props()['children'].value()`. `Child[]` 배열 반환. `?.` 없이 직접 접근하므로 `children` 프롭은 필수로 간주된다.

- `listTag` (`readonly`): `listStyle()`이 `'ordered'`이면 `'ol'`, `'unordered'`이면 `'ul'`, 그 외 모든 값(including `'none'`, `undefined`, 알 수 없는 값)은 `'div'`를 반환한다.

- `styleType` (`readonly`): `listStyle()`이 `'none'`이면 문자열 `'none'`을, 그 외에는 빈 문자열 `''`를 반환한다. `list-style-type` CSS 속성에 인라인 스타일로 바인딩된다.

**trackBy (`readonly` 화살표 함수 프로퍼티):**

시그니처: `(index: number, item: Child) => string`

`@for` 루프의 `track` 함수로 사용된다. `` `${item.basePath}/${item.id}` `` 복합 키 문자열을 반환해 Angular 변경 감지가 자식 항목을 안정적으로 재사용할 수 있도록 한다.

**템플릿 동작:**

`@switch (listTag())`로 세 가지 분기를 처리한다:
- `@case ('ol')`: `<ol [class]="'a2ui-list ' + direction()" [style.list-style-type]="styleType()">` 아래 `@for`로 `<li><a2ui-v09-component-host [componentKey]="child" [surfaceId]="surfaceId()">` 렌더링.
- `@case ('ul')`: `<ul ...>` 동일 구조.
- `@default`: `<div [class]="'a2ui-list ' + direction()" style="list-style-type: none;">` 아래 `@for`로 `<div class="a2ui-list-item-none"><a2ui-v09-component-host ...>` 렌더링. `div` 폴백의 경우 `list-style-type: none`을 인라인 스타일로 고정한다.

## 동작 흐름

1. `listStyle`, `direction`, `children`을 computed 시그널로 파생한다.
2. `listTag`가 `listStyle()` 값에 따라 `'ol'`, `'ul'`, `'div'` 중 하나를 결정한다.
3. 템플릿 `@switch`가 `listTag()` 값에 따라 적절한 HTML 목록 태그를 선택해 렌더링한다.
4. 각 자식 `Child` 항목을 `ComponentHostComponent`에 `componentKey`로 위임해 동적으로 렌더링한다.
5. `direction` 클래스(`vertical` 또는 `horizontal`)가 flex 방향을 결정한다.
6. `trackBy`로 변경 감지 시 자식 항목의 동일성을 `basePath/id` 복합 키로 추적한다.
