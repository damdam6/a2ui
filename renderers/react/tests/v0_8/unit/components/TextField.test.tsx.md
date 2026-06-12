# renderers/react/tests/v0_8/unit/components/TextField.test.tsx

## 개요

v0.8 React 렌더러의 `TextField` 컴포넌트에 대한 단위 테스트 파일이다. 기본 렌더링, 레이블 연결, 입력 타입 변환(`shortText`, `number`, `date`, `longText`), 사용자 인터랙션, 테마 컨테이너 구조를 검증한다. A2UI 스펙에서 `label` 필드는 필수이며 `text`, `type`, `validationRegexp`는 선택임을 주석으로 문서화하고 있다.

## 의존성

### 외부 패키지
- `vitest`: `describe`, `it`, `expect`
- `@testing-library/react`: `render`, `screen`, `fireEvent`
- `react`: `React`

### 저장소 내부 모듈
- [`../../utils`](../../utils/index.ts.md) — `TestWrapper`, `TestRenderer`, `createSimpleMessages`

## Exports

없음 (테스트 파일).

## 테스트 케이스 명세

모든 테스트는 `createSimpleMessages('tf-1', 'TextField', props)` 헬퍼로 메시지를 생성하고 `<TestWrapper><TestRenderer messages={...} /></TestWrapper>` 패턴으로 렌더링한다. `waitFor`를 사용하지 않는 동기 assertions을 사용한다.

### `describe('TextField Component')`

#### `describe('Basic Rendering')`

| 테스트 케이스 | 검증 동작 | props |
|---|---|---|
| `should render an input element` | `container.querySelector('input')` 존재 | `label.literalString='Username'` |
| `should render with wrapper div` | `.a2ui-textfield` 클래스를 가진 wrapper 엘리먼트 존재 | `label.literalString='Email'` |
| `should render with initial text value` | `input.value === 'John Doe'` | `label.literalString='Name'`, `text.literalString='John Doe'` |
| `should render placeholder text` | `input.placeholder === 'Please enter a value'` | `label.literalString='Search'` |

#### `describe('Label Rendering')`

| 테스트 케이스 | 검증 동작 |
|---|---|
| `should render label (required field)` | `screen.getByText('Username')` 존재 |
| `should associate label with input via htmlFor` | `label` 엘리먼트의 `for` 속성이 `input.id`와 일치 (접근성 연결) |

#### `describe('Input Types (type)')`

`textFieldType` prop에 따라 렌더되는 HTML 엘리먼트와 input type이 달라짐을 검증한다.

| 테스트 케이스 | props | 검증 |
|---|---|---|
| `should render text input by default (shortText)` | `type='shortText'` | `input.type === 'text'` |
| `should render number input for type=number` | `textFieldType='number'` | `input.type === 'number'` |
| `should render date input for type=date` | `textFieldType='date'` | `input.type === 'date'` |
| `should render textarea for type=longText` | `textFieldType='longText'` | `textarea` 존재, `input` 없음 |

#### `describe('User Interaction')`

`fireEvent.change`를 사용하여 사용자 입력을 시뮬레이션한다.

| 테스트 케이스 | 검증 동작 | 사용 이벤트 |
|---|---|---|
| `should update value on change` | `fireEvent.change` 후 `input.value === 'New value'` | `fireEvent.change(input, {target: {value: 'New value'}})` |
| `should update textarea value on change` | `fireEvent.change` 후 `textarea.value === 'Long text content'` | `textFieldType='longText'`, `fireEvent.change(textarea, ...)` |

#### `describe('Theme Support')`

| 테스트 케이스 | 검증 동작 |
|---|---|
| `should render within section container` | `<section>` 엘리먼트 존재, 그 안에 `input` 존재 |

## 동작 흐름

1. `createSimpleMessages`로 `surfaceUpdate` + `beginRendering` 메시지 쌍을 생성한다.
2. `<TestWrapper>`(= `A2UIProvider`)와 `<TestRenderer>`로 렌더링 컨텍스트를 구성한다.
3. `render()` 반환값의 `container` 또는 `screen` API로 DOM을 조회한다.
4. 인터랙션 테스트는 `fireEvent.change`로 값 변경 후 즉시 동기적으로 결과를 검증한다.
5. 이 파일에는 `waitFor`가 없으며 모든 assertions이 동기적임에 유의한다.
