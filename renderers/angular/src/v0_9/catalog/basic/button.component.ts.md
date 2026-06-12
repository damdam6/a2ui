# renderers/angular/src/v0_9/catalog/basic/button.component.ts

## 개요

A2UI v0.9 Button 컴포넌트의 Angular 구현이다. 단일 자식 컴포넌트(보통 Text)를 포함하는 클릭 가능한 `<button>` 엘리먼트를 렌더링한다. 클릭 시 `action` 프로퍼티에 정의된 액션을 `DataContext`를 통해 해석한 뒤 현재 서피스에 디스패치한다. `isValid` 프로퍼티가 `false`이면 버튼이 비활성화된다.

## 의존성

### 외부 패키지
- `@angular/core` — `Component`, `computed`, `ChangeDetectionStrategy`
- `@a2ui/web_core/v0_9` — `DataContext`
- `@a2ui/web_core/v0_9/basic_catalog` — `ButtonApi`

### 저장소 내부 모듈
- [`../../core/component-host.component`](../../core/component-host.component.ts.md)
- [`./basic-catalog-component`](./basic-catalog-component.ts.md)

## Exports

- `ButtonComponent` — Angular 스탠드얼론 컴포넌트 클래스

## 상세 명세

### `ButtonComponent`

`BasicCatalogComponent<typeof ButtonApi>`를 상속하는 Angular 컴포넌트.

- 셀렉터: `a2ui-v09-button`
- 변경 감지: `ChangeDetectionStrategy.OnPush`
- 임포트: `ComponentHostComponent`

#### computed 시그널

| 이름 | 반환 타입 | 로직 |
|---|---|---|
| `variant` | `string` | `props()['variant']?.value() ?? 'default'` |
| `child` | `Child \| undefined` | `props()['child']?.value()` |
| `action` | `Action \| undefined` | `props()['action']?.value()` |

#### `handleClick(): void`

버튼 클릭 이벤트 핸들러.

1. `action()`을 평가한다. 값이 없으면 아무것도 하지 않는다.
2. `surface()`를 평가한다. 값이 없으면 아무것도 하지 않는다.
3. `new DataContext(surface, this.dataContextPath())`로 데이터 컨텍스트 인스턴스를 생성한다.
4. `dataContext.resolveAction(action)`을 호출하여 액션을 해석한다.
5. `surface.dispatchAction(resolvedAction, this.componentId())`를 호출하여 `sourceComponentId`와 함께 액션을 디스패치한다.

#### 템플릿 로직

- `[type]`: `variant() === 'primary'`이면 `'submit'`, 아니면 `'button'`
- `[class]`: `'a2ui-button ' + variant()` — variant 이름이 CSS 클래스로 적용됨
- `[disabled]`: `props()['isValid']?.value() === false`
- `@if (child())`: child가 있을 때만 `a2ui-v09-component-host`를 렌더링

#### CSS 변수 (컴포넌트 내부 스타일)

| 변수 | 기본값 |
|---|---|
| `--a2ui-button-padding` | `var(--a2ui-spacing-m, 0.5rem) var(--a2ui-spacing-l, 1rem)` |
| `--a2ui-button-border-radius` | `var(--a2ui-spacing-s, 0.25rem)` |
| `--a2ui-button-border` | `var(--a2ui-border-width, 1px) solid var(--a2ui-color-border, #ccc)` |
| `--a2ui-button-margin` | `var(--a2ui-spacing-m, 0.5rem)` |
| `--a2ui-button-background` | `var(--a2ui-color-surface, #fff)` |
| `--a2ui-button-box-shadow` | `none` |
| `--a2ui-button-font-weight` | `normal` |

`.primary` 변형: 배경은 `var(--a2ui-color-primary, #17e)`, border 없음, 텍스트 색상 `var(--a2ui-color-on-primary, #fff)`.
`.borderless` 변형: 배경 없음, border 없음, padding 0, 텍스트 색상 `var(--a2ui-color-primary, #17e)`.
`:disabled` 상태: 배경 `#e9ecef`, 텍스트 `#6c757d`, border `#ced4da`, cursor `not-allowed`.

## 동작 흐름

1. Angular가 컴포넌트를 생성할 때 부모 클래스 생성자에서 `injectBasicCatalogStyles()`가 호출된다.
2. `props()` 시그널이 업데이트되면 `variant`, `child`, `action` computed 시그널이 무효화된다.
3. 사용자가 버튼을 클릭하면 `handleClick()`이 호출되고, 유효한 action과 surface가 있을 경우 DataContext를 통한 액션 해석 후 서피스에 디스패치된다.
