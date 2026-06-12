# renderers/lit/a2ui_explorer/tests/v0_9/09_login-form.test.ts

## 개요

A2UI 예제 `09_login-form.json`을 `<local-gallery>`에 로드하여 유효성 검사가 있는 로그인 폼 UI가 올바르게 렌더링되는지 검증하는 통합 테스트 파일이다. 제목, 부제목, 버튼 텍스트, 회원가입 유도 문구의 렌더링 여부만 확인하는 최소한의 스모크 테스트이다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈
- [`../utils/test-utils`](../utils/test-utils.ts.md) — `loadExample`, `getSurface`, `getDeepTextContent`
- [`../../src/local-gallery`](../../src/local-gallery.ts.md) — `LocalGallery` 타입

## Exports

없음 (테스트 파일)

## 테스트 케이스 명세

### `describe('Example: Login Form with Validation')`

**픽스처 설정:**
- `gallery: LocalGallery`, `surface: HTMLElement`, `textContent: string`을 스위트 범위 변수로 선언.
- `beforeEach`: `loadExample('09_login-form.json')`으로 갤러리를 마운트하고, `getSurface(gallery)`로 `a2ui-surface` 엘리먼트를 획득한 뒤, `getDeepTextContent(surface)`를 `textContent`에 저장.
- `afterEach`: `gallery?.remove()`로 DOM에서 갤러리를 제거.

---

**`it('should render text content')`**
- 검증 동작: 로그인 폼의 주요 텍스트 요소들이 모두 렌더링된다.
- 기대 결과: `textContent`에 `'Welcome back'`, `'Sign in to your account'`, `'Sign in'`, `"Don't have an account?"`, `'Sign up'` 포함.

## 동작 흐름

단일 테스트 케이스로 구성된 간단한 스모크 테스트이다. `beforeEach`에서 준비된 `textContent`를 사용하여 5개의 문자열이 포함되어 있는지 확인하는 것이 전부이며, 인터랙션이나 DOM 엘리먼트 직접 탐색은 없다.
