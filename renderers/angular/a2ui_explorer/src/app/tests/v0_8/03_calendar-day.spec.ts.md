# renderers/angular/a2ui_explorer/src/app/tests/v0_8/03_calendar-day.spec.ts

## 개요

v0.8 프로토콜의 "Calendar Day (basic)" 예제의 렌더링 및 버튼 인터랙션을 검증하는 Jasmine 통합 테스트 파일이다. 일정 텍스트(날짜, 이벤트 이름, 시간대)가 올바르게 표시되는지와, "Add to calendar" 버튼 및 "Discard" 버튼이 각각 `add`와 `discard` 액션을 디스패치하는지 확인한다.

## 의존성

### 외부 패키지
- `@angular/core/testing`: `ComponentFixture`

### 저장소 내부 모듈
- [`../../demo.component`](../../demo.component.ts.md) — `DemoComponent`
- [`../utils/test_utils`](../utils/test_utils.ts.md) — `Version`, `getCanvas`, `loadExample`, `wait`

## Exports

없음 (spec 파일).

## 테스트 케이스

**describe**: `'Example: Calendar Day (basic) (v0.8)'`

**공유 변수**: `textContent: string`, `fixture: ComponentFixture<DemoComponent>`

**`beforeEach` 설정**:
- `loadExample('Calendar Day (basic)', Version.V0_8)`로 `fixture`를 초기화한다. 예제 이름에 `(basic)` 접미사가 이미 포함되어 있어 `loadExample` 내부의 폴백 탐색 없이 직접 이름 매칭으로 찾는다.
- `getCanvas().textContent`를 `textContent`에 저장한다.

**테스트 1: `'should render expected text content'`**
- 검증: `textContent`에 다음 내용이 모두 포함되는지 확인한다.
  - 버튼 레이블: `'Add to calendar'`, `'Discard'`
  - 날짜 정보: `'Friday'`, `'28'`
  - 일정 이름 및 시간: `'Lunch'`와 `'12:00 - 12:45 PM'`, `'Q1 roadmap review'`와 `'1:00 - 2:00 PM'`, `'Team standup'`와 `'3:30 - 4:00 PM'`

**테스트 2: `'should dispatch add action on button click'`**
- 동작: `querySelectorAll('a2ui-button button')`으로 버튼 목록을 가져온다. 개수가 0보다 많은지 확인한 뒤 인덱스 0 버튼을 선택해 `.click()`을 호출하고 `fixture.detectChanges()` 후 `await wait(10)`으로 대기한다.
- 검증: `eventsLog.length > 0`이고 `eventsLog[0].action.name === 'add'`인지 확인한다.

**테스트 3: `'should dispatch discard action on button click'`**
- 동작: `querySelectorAll('a2ui-button button')`으로 버튼 목록을 가져온다. 개수가 1보다 많은지 확인한 뒤 인덱스 1 버튼을 선택해 `.click()`을 호출하고 `fixture.detectChanges()` 후 `await wait(10)`으로 대기한다.
- 검증: `eventsLog.length > 0`이고 `eventsLog[0].action.name === 'discard'`인지 확인한다.

**사용 픽스처/모킹**: `loadExample`이 TestBed를 구성하며 실제 v0.8 렌더러를 사용한다. 버튼 클릭 후 `wait(10)` ms 대기로 비동기 액션 처리를 보장한다.
