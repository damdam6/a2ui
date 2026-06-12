# renderers/lit/a2ui_explorer/tests/v0_9/32_advanced-form-validator.test.ts

## 개요

`32_advanced-form-validator.json` 예제를 로드하여 Advanced Form Validator 컴포넌트의 렌더링과 실시간 유효성 검사 동작을 검증하는 통합 테스트 파일이다. 초기 텍스트·날짜 표시, 입력 요소 수, 그리고 이메일·전화번호·우편번호에 잘못된 값을 입력했을 때 적절한 오류 메시지가 나타나는지를 확인한다.

## 의존성

### 외부 패키지
- (없음)

### 저장소 내부 모듈
- [`../utils/test-utils`](../utils/test-utils.ts.md) — `loadExample`, `getSurface`, `getDeepTextContent`, `querySelectorAllDeep`, `whenSettled` 임포트
- [`../../src/local-gallery`](../../src/local-gallery.ts.md) — `LocalGallery` 클래스 임포트

## Exports

명시적 export 없음.

## 동작 흐름

1. `beforeEach`: `loadExample('32_advanced-form-validator.json')` → `getSurface` → `getDeepTextContent`.
2. `afterEach`: `gallery.remove()`.
3. 유효성 검사 오류 테스트: `querySelectorAllDeep(surface, 'input')`으로 모든 입력 요소 조회 → `type`이 `'text'` 또는 미지정인 것만 필터링 → 특정 인덱스의 입력에 잘못된 값 설정 → `new Event('input')` 디스패치 → `whenSettled` → `textContent`에 오류 메시지가 포함되는지 확인.

## 테스트 케이스

### `should render text content`
- **검증 동작**: 텍스트에 `'Hello! Today is'`, `'Email Address'`, `'Phone Number'`, `'Zip Code'`, `'I agree to the terms and conditions'`, `'Submit Registration'`이 모두 포함되는지 확인한다.
- **픽스처**: `32_advanced-form-validator.json`

### `should render date`
- **검증 동작**: 텍스트에 `'December'`가 포함되는지 확인한다(컴포넌트가 날짜를 동적으로 표시함을 검증).

### `should render inputs`
- **검증 동작**: `input` 요소가 4개 이상 존재하는지 확인한다.

### `should show error for invalid email`
- **검증 동작**: 첫 번째 텍스트 입력(`textInputs[0]`)에 `'invalid-email'`을 입력하고 `input` 이벤트 디스패치 → `whenSettled` → 텍스트에 `'Invalid email format'`이 포함되는지 확인한다.
- **모킹/유틸**: `whenSettled(gallery)`.

### `should show error for invalid phone`
- **검증 동작**: 두 번째 텍스트 입력(`textInputs[1]`)에 `'123'` 입력 → `whenSettled` → `'Invalid phone format'` 포함 여부 확인.
- **모킹/유틸**: `whenSettled(gallery)`.

### `should show error for invalid zip`
- **검증 동작**: 세 번째 텍스트 입력(`textInputs[2]`)에 `'1234'` 입력 → `whenSettled` → `'Must be exactly 5 digits'` 포함 여부 확인.
- **모킹/유틸**: `whenSettled(gallery)`.
