# renderers/angular/a2ui_explorer/src/app/tests/v0_9/32_advanced-form-validator.spec.ts

## 개요

`Advanced Form Validator` 예제의 브라우저 통합 테스트 파일이다. 폼 렌더링 여부, 날짜 표시, 입력 필드 수, 그리고 이메일·전화번호·우편번호 각 필드에 잘못된 값을 입력했을 때 적절한 오류 메시지가 표시되는지 검증한다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture` 타입

### 저장소 내부 모듈
- [`../../demo.component`](../../demo.component.ts.md) — `DemoComponent`
- [`../utils`](../utils/index.ts.md) — `getCanvas`, `loadExample`, `wait`

## Exports

이 파일은 테스트 스위트 정의만 포함하며, 외부로 내보내는 항목이 없다.

## 테스트 케이스 명세

### 픽스처 및 셋업

`describe('Example: Advanced Form Validator')` 블록에서 `fixture: ComponentFixture<DemoComponent>`와 `textContent: string`을 공유 변수로 선언한다.

`beforeEach` — 각 테스트 전에 비동기로 실행된다.
1. `loadExample('Advanced Form Validator')`를 호출하여 예제를 로드하고 fixture를 얻는다.
2. `getCanvas().textContent`를 `textContent`에 저장한다 (`wait()` 없이 즉시 캡처).

### 테스트 케이스 1: `should render text content`

- **검증 대상**: 다음 텍스트들이 모두 캔버스에 존재해야 한다.
  - `'Hello! Today is'`
  - `'Email Address'`
  - `'Phone Number'`
  - `'Zip Code'`
  - `'I agree to the terms and conditions'`
  - `'Submit Registration'`

### 테스트 케이스 2: `should render date`

- **검증 대상**: 캔버스에 `'December'`라는 문자열이 존재해야 한다 (날짜 표시 검증).

### 테스트 케이스 3: `should render inputs`

- **검증 대상**: `fixture.nativeElement.querySelectorAll('input')`로 수집한 `<input>` 요소 수가 4개 이상이어야 한다.

### 테스트 케이스 4: `should show error for invalid email`

- **동작 시나리오**:
  1. `querySelectorAll('input')`으로 모든 입력을 수집한 뒤 `type === 'text'` 또는 `type` 없는 것만 필터링한다.
  2. `textInputs[0]` (첫 번째 텍스트 입력 = 이메일 필드)에 `'invalid-email'` 값을 설정하고 `'input'` 이벤트를 발생시킨다.
  3. `detectChanges()` → `wait(100)` → `detectChanges()` 순으로 변경을 반영한다.
- **검증 대상**: `textContent`에 `'Invalid email format'`이 포함되어야 한다.

### 테스트 케이스 5: `should show error for invalid phone`

- **동작 시나리오**:
  1. 이메일 테스트와 동일한 방식으로 텍스트 입력을 필터링한다.
  2. `textInputs[1]` (두 번째 텍스트 입력 = 전화번호 필드)에 `'123'` 값을 설정하고 `'input'` 이벤트를 발생시킨다.
  3. `detectChanges()` → `wait(100)` → `detectChanges()` 순으로 변경을 반영한다.
- **검증 대상**: `textContent`에 `'Invalid phone format'`이 포함되어야 한다.

### 테스트 케이스 6: `should show error for invalid zip`

- **동작 시나리오**:
  1. 이메일 테스트와 동일한 방식으로 텍스트 입력을 필터링한다.
  2. `textInputs[2]` (세 번째 텍스트 입력 = 우편번호 필드)에 `'1234'` 값을 설정하고 `'input'` 이벤트를 발생시킨다.
  3. `detectChanges()` → `wait(100)` → `detectChanges()` 순으로 변경을 반영한다.
- **검증 대상**: `textContent`에 `'Must be exactly 5 digits'`가 포함되어야 한다.

## 동작 흐름

`loadExample` → `getCanvas().textContent` 셋업 후, 기본 렌더링 검증(텍스트, 날짜, 입력 수)과 각 유효성 검사 필드별 오류 메시지 표시 검증을 순차적으로 수행한다. 유효성 검사 테스트는 실제 DOM 이벤트(`new Event('input')`)를 직접 발생시켜 동적 오류 표시를 테스트한다. input 필터링 조건 `type === 'text' || !i.type`으로 체크박스 등 다른 타입을 제외한다.
