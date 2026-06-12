# renderers/angular/src/v0_9/catalog/basic/choice-picker.component.ts

## 개요

A2UI v0.9 ChoicePicker 컴포넌트의 Angular 구현이다. 옵션 목록을 라디오 버튼/체크박스 또는 칩(chip) 형태로 렌더링하며, 단일 선택(`mutuallyExclusive`)과 다중 선택(`multipleSelection`) 두 가지 variant를 지원한다. 선택 변경 시 `onUpdate`를 통해 데이터 모델을 갱신한다.

## 의존성

### 외부 패키지
- `@angular/core` — `Component`, `computed`, `ChangeDetectionStrategy`
- `@a2ui/web_core/v0_9/basic_catalog` — `ChoicePickerApi`

### 저장소 내부 모듈
- [`./basic-catalog-component`](./basic-catalog-component.ts.md)

## Exports

- `ChoicePickerComponent` — Angular 스탠드얼론 컴포넌트 클래스

## 상세 명세

### `ChoicePickerComponent`

`BasicCatalogComponent<typeof ChoicePickerApi>`를 상속하는 Angular 컴포넌트.

- 셀렉터: `a2ui-v09-choice-picker`
- 변경 감지: `ChangeDetectionStrategy.OnPush`
- 임포트: 없음 (빈 배열)

#### computed 시그널

| 이름 | 반환 타입 | 로직 |
|---|---|---|
| `displayStyle` | `'checkbox' \| 'chips' \| undefined` | `props()['displayStyle']?.value()` |
| `options` | `{label: DynamicString; value: string}[]` | `props()['options']?.value() \|\| []` — undefined 시 빈 배열 반환 |
| `variant` | `'multipleSelection' \| 'mutuallyExclusive' \| undefined` | `props()['variant']?.value()` |
| `selectedValue` | `string[] \| undefined` | `props()['value']?.value()` |

#### `isMultiple(): boolean`

`variant() === 'multipleSelection'` 여부를 반환한다.

#### `isSelected(value: string): boolean`

`selectedValue()`가 배열이면 `Array.prototype.includes`로 포함 여부를 확인한다. 배열이 아니면 `selectedValue() === value`로 단순 비교한다.

#### `onCheckChange(value: string, event: Event): void`

체크박스/라디오 변경 핸들러. `event.target`을 `HTMLInputElement`로 캐스팅하여 `checked` 값을 얻고, `updateValue(value, checked)`를 호출한다.

#### `toggleActive(value: string): void`

칩 클릭 핸들러. `isSelected(value)`의 현재 상태를 반전시켜 `updateValue(value, !current)`를 호출한다.

#### `private updateValue(value: string, active: boolean): void`

선택 상태 업데이트의 핵심 로직.

다중 선택 모드(`isMultiple() === true`):
1. 현재 `selectedValue()`를 배열로 복사한다. 배열이 아니면 빈 배열로 초기화한다.
2. `active === true`이면 `value`가 목록에 없을 때만 추가한다.
3. `active === false`이면 `v !== value`인 항목만 남기도록 필터링한다.
4. `props()['value']?.onUpdate(next)`를 호출한다.

단일 선택 모드(`isMultiple() === false`):
1. `active === true`일 때만 `props()['value']?.onUpdate([value])`를 호출한다 (단일 값을 배열로 감싸서 전달).
2. `active === false`이면 아무것도 하지 않는다.

#### 템플릿 구조

`<div class="a2ui-choice-picker">` 안에 `displayStyle() === 'chips'` 분기:

- **chips**: `<div class="a2ui-chips-group">`에 `@for`로 각 옵션을 `<button class="a2ui-chip" [class.active]="isSelected(option.value)">`로 렌더링. 클릭 시 `toggleActive(option.value)` 호출.
- **기본(checkbox/radio)**: `<div class="a2ui-options-group">`에 `@for`로 각 옵션을 `<label class="a2ui-option-label">` + `<input [type]="isMultiple() ? 'checkbox' : 'radio'" [name]="componentId()" [value]="option.value" [checked]="isSelected(option.value)" (change)="onCheckChange(option.value, $event)">` + `<span>` 조합으로 렌더링.

#### CSS 변수

| 변수 | 기본값 |
|---|---|
| `--a2ui-choicepicker-gap` | `var(--a2ui-spacing-xs, 0.25rem)` |
| `--a2ui-choicepicker-padding` | `0` |
| `--a2ui-choicepicker-checkbox-size` | `1rem` |
| `--a2ui-choicepicker-chip-padding` | `var(--a2ui-spacing-s, 0.5rem) var(--a2ui-spacing-m, 1rem)` |
| `--a2ui-choicepicker-chip-border-radius` | `100px` |
| `--a2ui-choicepicker-chip-border` | `1px solid var(--a2ui-color-border, #ccc)` |
| `--a2ui-choicepicker-chip-background` | `var(--a2ui-color-surface, #fff)` |
| `--a2ui-choicepicker-chip-background-selected` | `var(--a2ui-color-primary, #17e)` |

`.a2ui-chip.active`: `background-color`과 `border-color`에 `--a2ui-choicepicker-chip-background-selected` 적용, 텍스트 색상 `var(--a2ui-color-on-primary, #fff)`.

## 동작 흐름

1. `props()` 업데이트 시 `displayStyle`, `options`, `variant`, `selectedValue` computed가 재평가된다.
2. `displayStyle`에 따라 템플릿이 칩 UI 또는 체크박스/라디오 UI를 선택적으로 렌더링한다.
3. 사용자 인터랙션(칩 클릭 또는 입력 변경)이 발생하면 `updateValue`가 호출되어 현재 선택 상태를 계산하고 `onUpdate`로 데이터 모델에 반영한다.
