# renderers/angular/a2ui_explorer/src/app/tests/v0_9/09_login-form.spec.ts

## 개요

`Login Form with Validation` 예제 컴포넌트의 텍스트 렌더링만을 검증하는 최소 구성의 Angular 통합 테스트 파일이다. `ComponentFixture`를 별도로 참조하지 않고 `loadExample`의 반환값도 저장하지 않으며, `getCanvas().textContent`만으로 UI 텍스트를 확인한다.

## 의존성

### 외부 패키지
없음.

### 저장소 내부 모듈
- [`../utils`](../utils/index.ts.md) — `getCanvas`, `loadExample`

## Exports

이 파일은 Jasmine `describe` 블록을 최상위에서 실행하며 별도로 export하는 항목은 없다.

## 테스트 케이스

### `describe('Example: Login Form with Validation')`

픽스처 변수: `textContent: string`

**`beforeEach`**
- `loadExample('Login Form with Validation')`를 비동기로 호출한다. 반환된 `fixture`는 저장하지 않는다.
- `getCanvas().textContent`로 캔버스 텍스트를 `textContent`에 저장한다.

---

**`should render text content`**
- 검증 동작: 캔버스 텍스트에 `'Welcome back'`, `'Sign in to your account'`, `'Sign in'`, `"Don't have an account?"`, `'Sign up'` 가 모두 포함되어 있는지 확인한다.
- 픽스처/모킹: `beforeEach`에서 로드된 `textContent` 사용.

## 동작 흐름

단일 `it` 블록으로만 구성된 간단한 구조다. `beforeEach`에서 예제를 로드하고 텍스트를 추출한 뒤, `should render text content` 테스트 케이스에서 로그인 폼의 핵심 텍스트 5개가 렌더링되었는지 일괄 검증한다.
