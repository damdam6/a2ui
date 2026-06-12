# renderers/angular/src/v0_9/catalog/basic/date-time-input.component.ts

## 개요

A2UI v0.9 DateTimeInput 컴포넌트의 Angular 구현체다. 날짜(`type="date"`)와 시간(`type="time"`) 입력 필드를 각각 조건부로 렌더링하며, 두 필드의 값을 결합해 ISO 형식(`YYYY-MM-DDTHH:mm:ss`) 문자열로 바인딩된 데이터 모델에 전달한다. 각 필드를 `enableDate` / `enableTime` 프롭으로 독립적으로 활성화하거나 비활성화할 수 있다.

## 의존성

### 외부 패키지
- `@angular/core` — `Component`, `computed`, `ChangeDetectionStrategy`
- `@a2ui/web_core/v0_9/basic_catalog` — `DateTimeInputApi`

### 저장소 내부 모듈
- [`./basic-catalog-component`](./basic-catalog-component.ts.md) — `BasicCatalogComponent`

## Exports

- `DateTimeInputComponent` — Angular standalone 컴포넌트 클래스

## 상세 명세

### `DateTimeInputComponent`

`BasicCatalogComponent<typeof DateTimeInputApi>`를 상속하는 Angular standalone 컴포넌트.

**데코레이터 설정:**
- `selector`: `'a2ui-v09-date-time-input'`
- `standalone: true`
- `imports`: `[]`
- `changeDetection`: `ChangeDetectionStrategy.OnPush`

**스타일 (컴포넌트 캡슐화):**

`.a2ui-date-time-container`는 `display: flex; flex-direction: column; gap: var(--a2ui-spacing-xs, 4px); width: 100%`. `.a2ui-date-time-label`은 `font-size: var(--a2ui-datetimeinput-label-font-size, var(--a2ui-label-font-size, var(--a2ui-font-size-s, 14px)))`, `font-weight: var(--a2ui-datetimeinput-label-font-weight, bold)`, 색상은 `var(--a2ui-text-color-text, var(--a2ui-color-on-background, #333))`. `.a2ui-date-time-inputs`는 `display: flex; gap: var(--a2ui-spacing-s, 8px); width: 100%`. `.a2ui-date-time-input`은 `padding: var(--a2ui-datetimeinput-padding, 8px)`, `border-radius: var(--a2ui-datetimeinput-border-radius, 4px)`, `border: var(--a2ui-datetimeinput-border, 1px solid #ccc)`, `background-color: var(--a2ui-datetimeinput-background, #fff)`, `color: var(--a2ui-datetimeinput-color, #333)`, `font-family: inherit`, `flex: 1`.

**computed 시그널:**

- `label` (`readonly`, 반환 타입 `string | undefined`):
  `props()['label']?.value()`를 반환한다. 템플릿에서 `@if (label())`로 조건부 렌더링된다. `undefined`이면 레이블 엘리먼트가 표시되지 않는다.

- `enableDate` (`readonly`, 반환 타입 `boolean`):
  `props()['enableDate']?.value() ?? true`. 프롭이 없거나 `undefined`이면 `true`가 기본값이므로 날짜 입력이 기본 활성화된다.

- `enableTime` (`readonly`, 반환 타입 `boolean`):
  `props()['enableTime']?.value() ?? false`. 프롭이 없거나 `undefined`이면 `false`가 기본값이므로 시간 입력이 기본 비활성화된다.

- `rawValue` (`private readonly`, 반환 타입 `string`):
  `props()['value']?.value() || ''`. ISO 형식 전체 문자열(`'2026-03-16T10:00:00'`), 날짜만 포함한 문자열(`'2026-03-16'`), 또는 빈 문자열이 될 수 있다.

- `dateValue` (`readonly`, 반환 타입 `string`):
  `rawValue()`가 빈 문자열이면 `''`를 반환한다. `'T'` 문자를 포함하면 `split('T')[0]`으로 날짜 부분(`YYYY-MM-DD`)만 추출한다. `'T'`가 없으면 `rawValue()` 전체를 반환한다.

- `timeValue` (`readonly`, 반환 타입 `string`):
  `rawValue()`가 빈 문자열이거나 `'T'`를 포함하지 않으면 `''`를 반환한다. `'T'`를 포함하면 `split('T')[1].substring(0, 5)`로 `HH:mm` 형식 5글자만 추출해 반환한다.

**메서드:**

- `handleDateChange(event: Event): void`
  - 접근 제어자: `public` (implicit)
  - `(event.target as HTMLInputElement).value`로 사용자가 선택한 새 날짜 문자열(`YYYY-MM-DD`)을 읽는다.
  - `enableTime()`이 `true`인 경우: `rawValue()`에 `'T'`가 있으면 `split('T')[1]`로 현재 시간 부분을 추출하고, 없으면 기본값 `'00:00:00'`을 사용한다. `` `${date}T${time}` `` 형태로 조합해 `props()['value']?.onUpdate(...)` 호출.
  - `enableTime()`이 `false`인 경우: 날짜 문자열만 그대로 `props()['value']?.onUpdate(date)` 호출.

- `handleTimeChange(event: Event): void`
  - 접근 제어자: `public` (implicit)
  - `(event.target as HTMLInputElement).value`로 사용자가 선택한 새 시간 문자열(`HH:mm`)을 읽는다.
  - `rawValue()`에 `'T'`가 있으면 `split('T')[0]`으로 현재 날짜 부분을 추출한다. `'T'`가 없으면 `rawValue()` 자체가 비어있지 않으면 그 값을, 비어있으면 `new Date().toISOString().split('T')[0]`으로 오늘 날짜를 사용한다.
  - `` `${date}T${time}:00` `` 형태로 조합해 `props()['value']?.onUpdate(...)` 호출. 입력값이 `HH:mm`이므로 `:00`을 붙여 초를 보완한다.

**템플릿 동작:**

최상위 `div.a2ui-date-time-container` 안에 레이블 영역과 입력 래퍼 `div.a2ui-date-time-inputs`가 배치된다. `@if (label())`로 `label.a2ui-date-time-label`을 조건부 렌더링한다. `@if (enableDate())`로 `input[type="date"]`를, `@if (enableTime())`로 `input[type="time"]`를 조건부 렌더링한다. 각 입력의 `[value]`는 `dateValue()` / `timeValue()`에, `(change)` 이벤트는 `handleDateChange` / `handleTimeChange`에 바인딩된다.

## 동작 흐름

1. `props()`에서 `label`, `enableDate`, `enableTime`, `value` 값을 computed 시그널로 파생한다.
2. `dateValue`, `timeValue`는 `rawValue()`를 `'T'` 기준으로 파싱해 날짜·시간 부분을 각각 computed로 분리한다.
3. 사용자가 날짜 입력을 변경하면 `handleDateChange`가 현재 시간 부분을 보존하며 ISO 문자열을 재조립해 `onUpdate`로 상위 데이터 모델에 전달한다.
4. 사용자가 시간 입력을 변경하면 `handleTimeChange`가 현재 날짜 부분을 보존하며 ISO 문자열을 재조립해 `onUpdate`로 상위 데이터 모델에 전달한다.
5. OnPush 변경 감지로 computed 시그널 값이 바뀔 때만 뷰를 업데이트한다.
