# renderers/react/tests/v0_8/unit/components/DateTimeInput.test.tsx

## 개요

A2UI 명세를 따르는 `DateTimeInput` 컴포넌트의 단위 테스트 모음이다. 필수 속성 `value`와 선택 속성 `enableDate`(기본값 `true`), `enableTime`(기본값 `false`)의 조합에 따른 input type 결정, 레이블 텍스트, 초기값 표시, 사용자 변경 이벤트 처리, DOM 구조를 검증한다.

## 의존성

### 외부 패키지
- `vitest` — `describe`, `it`, `expect`
- `@testing-library/react` — `render`, `fireEvent`
- `react` — `React`

### 저장소 내부 모듈
- [`../../utils`](../../utils/index.ts.md) — `TestWrapper`, `TestRenderer`, `createSimpleMessages`

## Exports

이 파일은 아무것도 export하지 않는다.

## 상세 명세 (테스트 케이스)

### describe: `DateTimeInput Component`

#### describe: `Basic Rendering`

- **`should render an input element`**
  - 검증: `value`를 `'2024-01-15'`로 설정한 `DateTimeInput`을 렌더링하면 `input` 요소가 DOM에 존재한다.

- **`should render with wrapper div`**
  - 검증: `.a2ui-datetime-input` 클래스를 가진 래퍼 요소가 DOM에 존재한다.

- **`should render with initial value`**
  - 검증: `value: {literalString: '2024-06-20'}`으로 설정하면 `input.value`가 `'2024-06-20'`이다.

- **`should render different values for different inputs`**
  - 검증: ID `'dt-1'`과 `'dt-2'`로 각각 `'2024-01-01'`과 `'2024-12-31'`을 값으로 갖는 두 컴포넌트를 독립 렌더링하면 각각의 input 값이 서로 다르고 올바른 초기값을 가진다.

#### describe: `Input Type`

`enableDate`와 `enableTime` 조합에 따라 input의 `type` 어트리뷰트가 결정되는 규칙을 검증한다.

- **`should render date input by default`** (`enableDate=true`, `enableTime=false`)
  - 검증: 속성 미지정(기본값) 시 `input.type`이 `'date'`이다. `'time'`도 `'datetime-local'`도 아니다.

- **`should render date input when enableDate=true`**
  - 검증: `enableDate: true`, `enableTime: false` → `input.type === 'date'`.

- **`should render time input when enableTime=true and enableDate=false`**
  - 검증: `enableDate: false`, `enableTime: true` → `input.type === 'time'`. `'date'`가 아니다.
  - value: `'14:30'`.

- **`should render datetime-local input when both enableDate and enableTime are true`**
  - 검증: `enableDate: true`, `enableTime: true` → `input.type === 'datetime-local'`. `'date'`도 `'time'`도 아니다.
  - value: `'2024-01-15T14:30'`.

- **`should render different input types for different configurations`**
  - 검증: date 전용 설정과 time 전용 설정을 각각 별도로 렌더링하면 두 input의 type이 서로 다르다 (`'date'` vs `'time'`).

#### describe: `Label`

- **`should render "Date" label for date input`**
  - 검증: `enableDate: true`, `enableTime: false` → `label.textContent === 'Date'`.

- **`should render "Time" label for time input`**
  - 검증: `enableDate: false`, `enableTime: true` → `label.textContent === 'Time'`.

- **`should render "Date & Time" label for datetime input`**
  - 검증: `enableDate: true`, `enableTime: true` → `label.textContent === 'Date & Time'`.

- **`should associate label with input via htmlFor`**
  - 검증: `label`의 `for` 어트리뷰트 값이 `input`의 `id`와 동일하다(접근성 연결).

#### describe: `User Interaction`

- **`should update value on change`**
  - 검증: 초기값 `'2024-01-15'`인 input에 `fireEvent.change`로 `'2024-06-20'`을 전달하면 `input.value`가 새 값으로 갱신되고 이전 값이 아니다.
  - 모킹: `fireEvent.change(input, {target: {value: '2024-06-20'}})`.

- **`should update time value on change`**
  - 검증: `enableTime: true`, 초기값 `'10:00'`인 time input에 `fireEvent.change`로 `'18:30'`을 전달하면 값이 갱신된다.

#### describe: `Structure`

- **`should have correct DOM structure: div > section > label + input`**
  - 검증: `.a2ui-datetime-input`의 태그명이 `'DIV'`이고, 그 안에 `section`이 있으며, `section`의 자식이 정확히 2개이고 첫 번째가 `LABEL`, 두 번째가 `INPUT`이다.

## 동작 흐름

각 테스트는 `createSimpleMessages(id, 'DateTimeInput', props)`를 사용해 단일 컴포넌트 메시지를 생성한다. `TestRenderer`가 메시지를 소비하여 React 렌더링을 수행하고 `TestWrapper`가 Context를 제공한다. 사용자 인터랙션 테스트는 `fireEvent`를 사용하여 DOM 이벤트를 발생시키고 `input.value`의 변화를 확인한다. 각 렌더링은 독립적인 `render()` 호출로 격리된다.
