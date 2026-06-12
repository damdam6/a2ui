# renderers/angular/src/v0_9/catalog/basic/check-box.component.ts

## 개요

A2UI v0.9 CheckBox 컴포넌트의 Angular 구현이다. 레이블과 함께 `<input type="checkbox">`를 렌더링하며, 체크 상태 변경 시 바인딩된 데이터 모델 프로퍼티를 `onUpdate` 콜백을 통해 업데이트한다. `accent-color` CSS 속성이 `--a2ui-color-primary`를 참조하여 테마 색상이 체크박스에 반영된다.

## 의존성

### 외부 패키지
- `@angular/core` — `Component`, `computed`, `ChangeDetectionStrategy`
- `@a2ui/web_core/v0_9/basic_catalog` — `CheckBoxApi`

### 저장소 내부 모듈
- [`./basic-catalog-component`](./basic-catalog-component.ts.md)

## Exports

- `CheckBoxComponent` — Angular 스탠드얼론 컴포넌트 클래스

## 상세 명세

### `CheckBoxComponent`

`BasicCatalogComponent<typeof CheckBoxApi>`를 상속하는 Angular 컴포넌트.

- 셀렉터: `a2ui-v09-check-box`
- 변경 감지: `ChangeDetectionStrategy.OnPush`
- 임포트: 없음 (빈 배열)

#### computed 시그널

| 이름 | 반환 타입 | 로직 |
|---|---|---|
| `value` | `boolean` | `props()['value']?.value() === true` — 엄격한 `true` 비교로 boolean 보장 |
| `label` | `string \| undefined` | `props()['label']?.value()` |

#### `handleChange(event: Event): void`

`(change)` 이벤트 핸들러.

1. `event.target`을 `HTMLInputElement`로 캐스팅하여 `.checked` 값을 읽는다.
2. `this.props()['value']?.onUpdate(checked)`를 호출하여 데이터 모델을 업데이트한다.

#### 템플릿 구조

`<label class="a2ui-check-box-label">` 안에 두 요소를 배치한다:
- `<input type="checkbox">`: `[checked]="value()"`, `(change)="handleChange($event)"`, `class="a2ui-check-box-input"`
- `<span class="a2ui-check-box-text">`: `{{ label() }}` 텍스트 표시

#### CSS 변수

| 변수 | 기본값 |
|---|---|
| `--a2ui-checkbox-margin` | `var(--a2ui-spacing-m, 16px)` |
| `--a2ui-checkbox-gap` | `var(--a2ui-spacing-s, 0.5rem)` |
| `--a2ui-checkbox-label-font-size` | `var(--a2ui-label-font-size, var(--a2ui-font-size-s, 16px))` |
| `--a2ui-checkbox-label-font-weight` | `bold` |
| `--a2ui-checkbox-size` | `1rem` (width/height 모두 적용) |
| `--a2ui-checkbox-background` | `inherit` |
| `--a2ui-checkbox-border` | `var(--a2ui-border-width, 1px) solid #ccc` |
| `--a2ui-checkbox-border-radius` | `4px` |

`accent-color`는 `var(--a2ui-color-primary)`로 고정되어 테마 primary 색상이 체크박스에 적용된다.

## 동작 흐름

1. 부모 클래스 생성자에서 basic catalog 스타일이 주입된다.
2. `props()` 변경 시 `value`와 `label` computed가 재평가된다.
3. 사용자가 체크박스를 클릭하면 DOM의 `change` 이벤트가 발생하고, `handleChange`가 새로운 checked 상태를 `onUpdate`로 전달하여 데이터 모델을 갱신한다.
