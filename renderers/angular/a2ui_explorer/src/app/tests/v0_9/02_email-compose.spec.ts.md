# renderers/angular/a2ui_explorer/src/app/tests/v0_9/02_email-compose.spec.ts

## 개요

v0.9 버전의 "Email Compose" 예제가 Angular 렌더러에서 올바르게 렌더링되고, "Send email" 및 "Discard" 버튼 클릭 시 각각 `send`와 `discard` 액션이 `eventsLog`에 기록되는지 검증하는 Jasmine 테스트 스위트다. 3개의 테스트로 구성되며, 버튼을 `textContent` 포함 여부로 식별하는 패턴을 사용한다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture` 타입

### 저장소 내부 모듈
- [`../../demo.component`](../../demo.component.ts.md) — `DemoComponent` 클래스 (eventsLog 상태 보유)
- [`../utils`](../utils/index.ts.md) — `getCanvas`, `loadExample`, `wait` 유틸리티 함수 제공

## Exports

이 파일은 Jasmine `describe` 블록을 최상위에 선언하며, TypeScript 모듈 export는 없다.

## 테스트 케이스 상세 명세

### `describe`: `'Example: Email Compose'`

**픽스처 / 셋업**
- `fixture: ComponentFixture<DemoComponent>` — `loadExample` 반환값.
- `component: DemoComponent` — `fixture.componentInstance`.
- `textContent: string` — 각 테스트 전에 캔버스 DOM의 텍스트를 저장하는 변수.
- `beforeEach`: `loadExample('Email Compose')`를 `await`로 호출하고, `fixture.componentInstance`를 `component`에 할당하며, `getCanvas().textContent`를 저장한다.

---

#### 테스트 1: `'should render text content'`

캔버스 텍스트에 다음이 모두 포함되는지 확인한다.
- 헤더 레이블: `'FROM'`, `'TO'`, `'SUBJECT'`
- 버튼 레이블: `'Send email'`, `'Discard'`
- 이메일 주소: `'alex@acme.com'`, `'jordan@acme.com'`
- 제목: `'Q4 Revenue Forecast'`
- 본문: `'Hi Jordan,'`, `'Following up on our call. Please review the attached Q4 forecast and let me know if you have questions before the board meeting.'`, `'Best,'`, `'Alex'`

---

#### 테스트 2: `'should handle Send button click'`

1. `querySelectorAll('.a2ui-button')`로 모든 버튼 요소를 `HTMLButtonElement[]`로 스프레드(`[...]`)하고, `Array.find`로 `b.textContent.includes('Send')`를 만족하는 `sendBtn`을 찾는다. `!` non-null assertion 적용. `withContext('Should find Send button')`와 함께 존재 여부를 검증한다.
2. `sendBtn.click()`, `fixture.detectChanges()`, `await wait(50)`, `fixture.detectChanges()` 순서로 실행한다.
3. `component.eventsLog.length === 1`이고 `eventsLog[0].action.name === 'send'`인지 검증한다.

---

#### 테스트 3: `'should handle Discard button click'`

Send 버튼 테스트와 동일한 패턴으로, `b.textContent.includes('Discard')`로 `discardBtn`을 찾는다. `withContext('Should find Discard button')`와 함께 검증 후 클릭하고, `eventsLog[0].action.name === 'discard'`인지 확인한다.

**참고**: 각 `beforeEach`에서 `eventsLog`가 초기화되므로 버튼 클릭 후 `length === 1`을 기대한다.

## 동작 흐름

`beforeEach` → 예제 로드 및 컴포넌트/텍스트 스냅샷 → 테스트 1(모든 텍스트 요소 확인) → 테스트 2(Send 버튼 텍스트 기반 식별 → 클릭 → 50ms 대기 → `eventsLog[0].action.name === 'send'` 검증) → 테스트 3(Discard 버튼 동일 패턴 → `'discard'` 검증).
