# renderers/angular/src/v0_9/catalog/basic/divider.component.ts

## 개요

A2UI v0.9 Divider 컴포넌트의 Angular 구현체다. 콘텐츠를 구분하는 수평 또는 수직 선을 `<hr>` 엘리먼트로 렌더링한다. `axis` 프롭 값에 따라 호스트 엘리먼트의 `width` 스타일을 동적으로 설정하고, `<hr>`에 `horizontal` 또는 `vertical` CSS 클래스를 조건부 적용해 방향별 스타일을 제어한다.

## 의존성

### 외부 패키지
- `@angular/core` — `Component`, `computed`, `ChangeDetectionStrategy`
- `@a2ui/web_core/v0_9/basic_catalog` — `DividerApi`

### 저장소 내부 모듈
- [`./basic-catalog-component`](./basic-catalog-component.ts.md) — `BasicCatalogComponent`

## Exports

- `DividerComponent` — Angular standalone 컴포넌트 클래스

## 상세 명세

### `DividerComponent`

`BasicCatalogComponent<typeof DividerApi>`를 상속하는 Angular standalone 컴포넌트.

**데코레이터 설정:**
- `selector`: `'a2ui-v09-divider'`
- `standalone: true`
- `imports`: `[]`
- `changeDetection`: `ChangeDetectionStrategy.OnPush`
- `host` 바인딩:
  - `[style.display]`: 항상 리터럴 `"block"`
  - `[style.width]`: `axis() === 'horizontal'`이면 `"100%"`, 그렇지 않으면 `"auto"`

**스타일 (컴포넌트 캡슐화):**

`.a2ui-divider`는 `border: 0`, `border-top: var(--a2ui-divider-border, var(--a2ui-border-width, 1px) solid var(--a2ui-color-border, #ccc))`, `margin: var(--a2ui-divider-spacing, var(--a2ui-spacing-m, 16px)) 0`, `width: 100%`가 기본이다.

`.a2ui-divider.vertical` 클래스가 추가되면:
- `width: var(--a2ui-border-width, 1px)`, `height: 100%`로 변경.
- `margin: 0 var(--a2ui-divider-spacing, var(--a2ui-spacing-m, 16px))`로 마진 방향 전환.
- `border-top: 0`으로 수평 선을 제거하고, `border-left: var(--a2ui-divider-border, ...)` 적용.

**computed 시그널:**

- `axis` (`readonly`, 반환 타입 `string`):
  `props()['axis']?.value() ?? 'horizontal'`. `axis` 프롭이 없거나 `undefined`이면 기본값 `'horizontal'`을 반환한다.

**템플릿 동작:**

`<hr class="a2ui-divider" [class.horizontal]="axis() === 'horizontal'" [class.vertical]="axis() === 'vertical'">` 단일 엘리먼트를 렌더링한다. `axis()` 값에 따라 정확히 하나의 방향 클래스만 적용된다.

## 동작 흐름

1. `props()`에서 `axis`를 computed 시그널로 파생한다. 기본값은 `'horizontal'`.
2. 호스트 엘리먼트의 `width` 스타일은 `axis()`가 `'horizontal'`이면 `100%`, 그 외이면 `auto`로 반응형 설정된다.
3. 내부 `<hr>` 엘리먼트에 `horizontal` 또는 `vertical` CSS 클래스가 적용되어 방향별 스타일(선의 방향, 마진, 크기)이 결정된다.
