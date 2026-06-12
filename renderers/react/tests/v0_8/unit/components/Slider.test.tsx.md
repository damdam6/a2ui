# renderers/react/tests/v0_8/unit/components/Slider.test.tsx

## 개요

`Slider` 컴포넌트의 A2UI 명세 준수 여부를 검증하는 단위 테스트 파일이다. 렌더링 구조, `min`/`max`/`value` 속성 처리, 레이블 유무에 따른 DOM 구조 차이, 사용자 드래그 인터랙션 후 값 동기화까지 포괄적으로 검증한다. `createSimpleMessages` 헬퍼를 사용해 단순화된 메시지 구성을 취한다.

## 의존성

### 외부 패키지
- `vitest` — `describe`, `it`, `expect`
- `@testing-library/react` — `render`, `fireEvent`
- `react` — `React`

### 저장소 내부 모듈
- [`../../utils`](../../utils/index.ts.md) — `TestWrapper`, `TestRenderer`, `createSimpleMessages`

## Exports

이 파일은 아무것도 export하지 않는다. Vitest 테스트 스위트 파일이다.

## 상세 명세 — 테스트 케이스

### 헬퍼 / 픽스처 패턴

모든 테스트는 `createSimpleMessages(id, componentType, props)` 헬퍼를 사용한다. 이 함수는 주어진 ID와 컴포넌트 타입(`'Slider'`), props 객체로 `ServerToClientMessage[]` 배열을 생성한다.

Slider 컴포넌트의 props 구조:
- `value`: `{ literalNumber: number }` (필수)
- `minValue`: `number` (선택, 미지정 시 기본값 `0`)
- `maxValue`: `number` (선택, 미지정 시 기본값 `0`)
- `label`: `{ literalString: string }` (선택)

---

### `describe('Basic Rendering')`

#### `should render a range input`
- `value: { literalNumber: 50 }` 인 Slider를 렌더링한다.
- `input[type="range"]`가 DOM에 존재하고, `tagName`이 `'INPUT'`이며, `type` 속성이 `'range'`임을 검증한다.

#### `should render with wrapper div having correct class`
- 동일 구성으로 렌더링한다.
- `.a2ui-slider` 클래스를 가진 `DIV` 엘리먼트가 DOM에 존재함을 검증한다.

#### `should render with exact initial value`
- `value: 75`, `minValue: 0`, `maxValue: 100`으로 렌더링한다.
- `input.value`가 문자열 `'75'`임을 검증한다. 기본값(`'50'`, `'0'`)이 아님을 추가로 확인한다.

#### `should display current value in span matching input value`
- `value: 42`, `minValue: 0`, `maxValue: 100`으로 렌더링한다.
- `container.querySelector('section span')`의 `textContent`가 `'42'`이고, `input.value`도 동일하게 `'42'`임을 검증한다.

#### `should render different values for different inputs`
- `value: 25`인 Slider와 `value: 75`인 Slider를 각각 독립 렌더링한다.
- 두 input의 value가 각각 `'25'`, `'75'`이고 서로 다름을 검증한다.

---

### `describe('Min/Max Values')`

#### `should set exact min attribute value`
- `minValue: 10`, `maxValue: 100`으로 렌더링한다.
- `input.min`이 `'10'`임을 검증한다. 기본값 `'0'`이 아님을 추가로 확인한다.

#### `should set exact max attribute value`
- `minValue: 0`, `maxValue: 200`으로 렌더링한다.
- `input.max`가 `'200'`임을 검증한다. HTML 기본값 `'100'`이 아님을 추가로 확인한다.

#### `should default min/max to 0 when not provided`
- `value: 0`만 지정하고 `minValue`/`maxValue`를 전달하지 않는다.
- `input.min`과 `input.max` 모두 `'0'`임을 검증한다.

#### `should handle negative min values`
- `value: 0`, `minValue: -100`, `maxValue: 100`으로 렌더링한다.
- `input.min`이 `'-100'`임을 검증한다.

---

### `describe('User Interaction')`

#### `should update input value on change`
- `value: 50`으로 초기 렌더링 후 `fireEvent.change(input, { target: { value: '80' } })`를 발행한다.
- 초기 `input.value`가 `'50'`이고, 이벤트 후 `'80'`이 되며, 구 값 `'50'`이 아님을 검증한다.

#### `should update displayed span value in sync with input`
- `value: 50`으로 초기 렌더링 후 `fireEvent.change(input, { target: { value: '25' } })`를 발행한다.
- 이벤트 전 `span.textContent`가 `'50'`, 이벤트 후 `input.value`와 `span.textContent` 모두 `'25'`임을 검증한다(동기적 상태 연동 확인).

#### `should handle multiple sequential value changes`
- `value: 50`으로 초기 렌더링 후 `fireEvent.change`를 세 번 순차 발행한다: `'10'` → `'90'` → `'50'`.
- 각 변경 후 `input.value`와 `span.textContent`가 동일한 값을 나타냄을 검증한다.

---

### `describe('Structure')`

#### `should have correct DOM structure: label, input, span in order`
- `label: { literalString: 'Volume' }`, `value: 50`으로 렌더링한다.
- `section` 내 직계 자식이 3개이고, 순서대로 `LABEL`, `INPUT`, `SPAN` 태그임을 검증한다.

#### `should omit label from DOM structure when not provided`
- `label` 미지정, `value: 50`으로 렌더링한다.
- `section` 내 직계 자식이 2개이고, 순서대로 `INPUT`, `SPAN` 태그임을 검증한다(레이블 없으면 3개 아닌 2개).

#### `should have input inside section container`
- `value: 50`으로 렌더링한다.
- `section` 내부에 `input[type="range"]`가 존재함을 검증한다.

## 동작 흐름

1. `createSimpleMessages`로 단일 Slider 컴포넌트를 포함하는 메시지 배열을 생성한다.
2. `TestWrapper > TestRenderer`가 메시지를 처리해 DOM에 Slider를 렌더링한다.
3. 렌더링 결과는 `.a2ui-slider > section > [label?] input[type=range] span` 구조를 가진다.
4. label prop 존재 여부에 따라 section 직계 자식 수가 2(없을 때) 또는 3(있을 때)으로 달라진다.
5. `fireEvent.change`로 슬라이더 값 변경 이벤트를 시뮬레이션하며, 컴포넌트 내부 상태가 업데이트되어 `input.value`와 `span.textContent`가 동기화됨을 확인한다.
6. `minValue`/`maxValue` 미지정 시 두 속성 모두 `0`으로 설정되는 경계 케이스를 검증한다.
