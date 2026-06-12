# renderers/angular/a2ui_explorer/src/app/tests/v0_9/17_event-detail.spec.ts

## 개요

`Event Detail` 예제 컴포넌트의 렌더링 및 버튼 인터랙션을 검증하는 Angular 통합 테스트 파일이다. 이벤트 제목·위치·설명·날짜·아이콘 렌더링을 확인하고, `'Accept'` 버튼과 `'Decline'` 버튼을 각각 클릭했을 때 `eventsLog`에 `'accept'`와 `'decline'` 액션이 기록되는지 검증한다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture` 타입

### 저장소 내부 모듈
- [`../../demo.component.ts`](../../demo.component.ts.md) — `DemoComponent`
- [`../utils`](../utils/index.ts.md) — `getCanvas`, `loadExample`, `wait`

## Exports

이 파일은 Jasmine `describe` 블록을 최상위에서 실행하며 별도로 export하는 항목은 없다.

## 테스트 케이스

### `describe('Example: Event Detail')`

픽스처 변수: `fixture: ComponentFixture<DemoComponent>`, `component: DemoComponent`, `textContent: string`

**`beforeEach`**
- `loadExample('Event Detail')`를 비동기로 호출하여 `fixture`를 얻는다.
- `fixture.componentInstance`를 `component`에 저장한다.
- `getCanvas().textContent`를 `textContent`에 저장한다.

---

**`should render text content`**
- 검증 동작: 텍스트에 `'Product Launch Meeting'`, `'Conference Room A, Building 2'`, `'Review final product specs and marketing materials before the Q1 launch.'`, `'Accept'`, `'Decline'`, `'Dec 19'` 가 포함되어 있는지 확인한다. `'Dec 19'`는 UTC 기준 날짜 렌더링으로 "approximate or specific if we trust UTC"라는 주석이 있다.
- 픽스처/모킹: `textContent` 사용.

---

**`should render icons`**
- 검증 동작: `querySelectorAll('.a2ui-icon')`로 아이콘 요소 배열을 수집하고 길이가 2 이상인지 확인한다. 텍스트에 `'calendar_today'`와 `'location_on'` 아이콘 리터럴이 포함되어 있는지 검증한다.
- 픽스처/모킹: `fixture.nativeElement` DOM 쿼리 및 `textContent`.

---

**`should handle Accept button click`**
- 검증 동작: `.a2ui-button` 배열에서 `'Accept'`를 포함하는 버튼을 찾아 클릭한다. `fixture.detectChanges()` + `wait(50)` + `fixture.detectChanges()` 후 `component.eventsLog.length`가 `1`이고, `eventsLog[0].action.name`이 `'accept'`인지 검증한다.
- 픽스처/모킹: `fixture.nativeElement` DOM 쿼리, `wait(50)` 비동기 대기, `component.eventsLog` 상태 확인.

---

**`should handle Decline button click`**
- 검증 동작: `.a2ui-button` 배열에서 `'Decline'`을 포함하는 버튼을 찾아 클릭한다. 동일한 변경 감지 시퀀스 후 `eventsLog[0].action.name`이 `'decline'`인지 검증한다.
- 픽스처/모킹: `fixture.nativeElement` DOM 쿼리, `wait(50)` 비동기 대기, `component.eventsLog` 상태 확인.

## 동작 흐름

`beforeEach`에서 예제를 로드하고 컴포넌트 인스턴스를 저장한다. 렌더링 테스트 두 개는 텍스트와 아이콘(두 개 이상)을 정적으로 검증하고, 인터랙션 테스트 두 개는 각각 Accept와 Decline 버튼의 클릭 이벤트와 그 결과 액션 이름을 독립적으로 검증한다.
