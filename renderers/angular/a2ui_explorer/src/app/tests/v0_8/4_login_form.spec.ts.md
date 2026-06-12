# renderers/angular/a2ui_explorer/src/app/tests/v0_8/4_login_form.spec.ts

## 개요

v0.8 버전의 "Login Form (minimal)" 예제가 Angular 렌더러에서 올바르게 렌더링되고, 폼 입력 후 제출 시 `login_submitted` 액션이 사용자 입력값과 함께 `eventsLog`에 기록되는지 검증하는 Jasmine 테스트 스위트다. 2개의 `<input>` 필드에 직접 값을 설정하고 네이티브 `input` 이벤트를 디스패치하는 폼 인터랙션 패턴을 사용한다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture` 타입

### 저장소 내부 모듈
- [`../../demo.component`](../../demo.component.ts.md) — `DemoComponent` 클래스 (eventsLog 상태 보유)
- [`../utils/test_utils`](../utils/test_utils.ts.md) — `Version`, `getCanvas`, `loadExample`, `wait` 유틸리티 함수 제공

## Exports

이 파일은 Jasmine `describe` 블록을 최상위에 선언하며, TypeScript 모듈 export는 없다.

## 테스트 케이스 상세 명세

### `describe`: `'Example: Login Form (minimal) (v0.8)'`

**픽스처 / 셋업**
- `textContent: string` — 각 테스트 전에 캔버스 DOM의 텍스트를 저장하는 변수.
- `fixture: ComponentFixture<DemoComponent>` — `loadExample` 반환값.
- `beforeEach`: `loadExample('Login Form (minimal)', Version.V0_8)`를 `await`로 호출해 `fixture`를 얻고, `getCanvas().textContent`를 저장한다.

---

#### 테스트 1: `'should render expected text content'`

캔버스 텍스트에 `'Login'`과 `'Sign In'`이 포함되는지 확인한다.

---

#### 테스트 2: `'should dispatch login_submitted action with the form contents on button click'`

1. `fixture.componentInstance`로 `component` 참조를 얻는다.
2. `querySelectorAll('input')`으로 입력 요소를 조회하고 2개 이상임을 검증한다(`toBeGreaterThanOrEqual(2)`).
3. `inputs[0].value = 'testuser'`로 값을 설정하고 `new Event('input')`을 `dispatchEvent`로 발행해 Angular 바인딩을 트리거한다.
4. `inputs[1].value = 'testpass'`로 값을 설정하고 동일하게 `input` 이벤트를 발행한다.
5. `fixture.detectChanges()`를 호출한다.
6. `querySelectorAll('a2ui-button button')`으로 버튼을 조회하고 첫 번째 버튼(`btn`)이 `truthy`인지 확인한 뒤 `.click()`으로 클릭한다.
7. `fixture.detectChanges()` 후 `await wait(10)`으로 10ms 대기한다.
8. `component.eventsLog.length`가 0보다 크고, `eventsLog[0].action.name`이 `'login_submitted'`이며, `eventsLog[0].action.context`가 `{ user: 'testuser', pass: 'testpass' }`와 `toEqual`로 깊은 동등성을 만족하는지 검증한다.

**검증 포인트**: 액션 컨텍스트에 입력된 사용자명(`user`)과 비밀번호(`pass`)가 정확히 담겨야 한다.

## 동작 흐름

`beforeEach` → 예제 로드 → 텍스트 스냅샷 → 테스트 1(정적 텍스트) → 테스트 2(입력값 세팅 → `input` 이벤트 발행 → `detectChanges` → 버튼 클릭 → 10ms 대기 → `eventsLog` 내 `login_submitted` 액션 및 `context` 값 검증).
