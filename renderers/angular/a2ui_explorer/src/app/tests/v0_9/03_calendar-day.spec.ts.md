# renderers/angular/a2ui_explorer/src/app/tests/v0_9/03_calendar-day.spec.ts

## 개요

v0.9 버전의 "Calendar Day" 예제가 Angular 렌더러에서 올바르게 렌더링되는지 검증하는 Jasmine 테스트 스위트다. 일정 목록(이벤트명, 시간대)과 버튼 텍스트 확인, 그리고 날짜 숫자·요일명이 표시되는지를 2개의 테스트로 검증한다. 날짜 관련 검증은 동적 값을 허용하는 OR 조건 로직을 사용한다.

## 의존성

### 외부 패키지
없음 (Jasmine은 테스트 런타임으로 암묵적 제공)

### 저장소 내부 모듈
- [`../utils`](../utils/index.ts.md) — `getCanvas`, `loadExample` 유틸리티 함수 제공

## Exports

이 파일은 Jasmine `describe` 블록을 최상위에 선언하며, TypeScript 모듈 export는 없다.

## 테스트 케이스 상세 명세

### `describe`: `'Example: Calendar Day'`

**픽스처 / 셋업**
- `textContent: string` — 각 테스트 전에 캔버스 DOM의 텍스트를 저장하는 변수.
- `beforeEach`: `loadExample('Calendar Day')`를 `await`로 호출한 뒤, `getCanvas().textContent`를 저장한다.

---

#### 테스트 1: `'should render text content'`

캔버스 텍스트에 다음이 모두 포함되는지 확인한다.
- 일정명: `'Lunch'`, `'Q1 roadmap review'`, `'Team standup'`
- 시간대: `'12:00 - 12:45 PM'`, `'1:00 - 2:00 PM'`, `'3:30 - 4:00 PM'`
- 버튼: `'Add to calendar'`, `'Discard'`

---

#### 테스트 2: `'should render date and day name'`

1. **날짜 숫자**: `textContent.includes('28') || textContent.includes('27')` OR 조건으로 `hasDayNumber`를 계산하고 `withContext('Should render day number')`와 함께 `toBeTruthy()`를 검증한다.
2. **요일명**: `textContent`에 `'Sunday'`, `'Monday'`, `'Tuesday'`, `'Wednesday'`, `'Thursday'`, `'Friday'`, `'Saturday'` 중 하나가 포함되는지 OR 조건으로 `hasDayName`을 계산하고 `withContext('Should render day name')`와 함께 `toBeTruthy()`를 검증한다.

**경계 케이스**: 날짜 숫자와 요일명은 실행 시점에 따라 동적으로 계산될 수 있어 복수 후보 값을 OR로 허용한다.

## 동작 흐름

`beforeEach` → `loadExample`로 예제 초기화 → 텍스트 스냅샷 → 테스트 1(고정 일정/시간/버튼 텍스트 검증) → 테스트 2(동적 날짜 숫자 및 요일명 후보 OR 검증).
