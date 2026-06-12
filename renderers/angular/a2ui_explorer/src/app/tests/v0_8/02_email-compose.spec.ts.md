# renderers/angular/a2ui_explorer/src/app/tests/v0_8/02_email-compose.spec.ts

## 개요

v0.8 프로토콜의 "Email Compose" 예제의 렌더링 및 버튼 인터랙션을 검증하는 Jasmine 통합 테스트 파일이다. 이메일 헤더·본문 텍스트 렌더링을 확인하고, "Send email" 버튼과 "Discard" 버튼 클릭 시 각각 `send`와 `discard` 액션이 이벤트 로그에 기록되는지 검증한다.

## 의존성

### 외부 패키지
- `@angular/core/testing`: `ComponentFixture`

### 저장소 내부 모듈
- [`../../demo.component`](../../demo.component.ts.md) — `DemoComponent`
- [`../utils/test_utils`](../utils/test_utils.ts.md) — `Version`, `getCanvas`, `loadExample`, `wait`

## Exports

없음 (spec 파일).

## 테스트 케이스

**describe**: `'Example: Email Compose (v0.8)'`

**공유 변수**: `textContent: string`, `fixture: ComponentFixture<DemoComponent>`

**`beforeEach` 설정**:
- `loadExample('Email Compose', Version.V0_8)`로 `fixture`를 초기화한다.
- `getCanvas().textContent`를 `textContent`에 저장한다.

**테스트 1: `'should render email headers'`**
- 검증: `textContent`에 `'FROM'`, `'alex@acme.com'`, `'TO'`, `'jordan@acme.com'`, `'SUBJECT'`, `'Q4 Revenue Forecast'`가 모두 포함되는지 확인한다.

**테스트 2: `'should render email body'`**
- 검증: `textContent`에 `'Hi Jordan,'`, `'Following up on our call'`, `'Best,'`, `'Alex'`가 모두 포함되는지 확인한다.

**테스트 3: `'should render buttons'`**
- 검증: `textContent`에 `'Send email'`과 `'Discard'`가 모두 포함되는지 확인한다.

**테스트 4: `'should dispatch send action on button click'`**
- 동작: `fixture.nativeElement.querySelector('a2ui-button button')`으로 첫 번째 버튼을 선택한다. 존재 여부를 확인한 뒤 `.click()`을 호출하고 `fixture.detectChanges()`를 실행한다. `await wait(10)`으로 비동기 처리를 기다린다.
- 검증: `component.eventsLog.length > 0`이고 `eventsLog[0].action.name === 'send'`인지 확인한다.

**테스트 5: `'should dispatch discard action on button click'`**
- 동작: `fixture.nativeElement.querySelectorAll('a2ui-button button')`로 모든 버튼 목록을 가져온 뒤 개수가 1보다 많은지 확인한다. 인덱스 1(두 번째 버튼)에 `.click()`을 호출하고 `fixture.detectChanges()` 및 `await wait(10)`으로 대기한다.
- 검증: `eventsLog.length > 0`이고 `eventsLog[0].action.name === 'discard'`인지 확인한다.

**사용 픽스처/모킹**: `loadExample`이 TestBed를 구성하며 실제 v0.8 렌더러를 사용한다. 버튼 클릭 후 `wait(10)` ms 대기로 비동기 액션 디스패치 처리를 보장한다.
