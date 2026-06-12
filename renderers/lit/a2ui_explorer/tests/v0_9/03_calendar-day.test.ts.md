# renderers/lit/a2ui_explorer/tests/v0_9/03_calendar-day.test.ts

## 개요

A2UI 예제 `03_calendar-day.json`을 `<local-gallery>`에 로드하여 하루 일정 카드 UI가 올바르게 렌더링되는지 검증하는 통합 테스트 파일이다. 일정 항목(제목, 시간대), 버튼 레이블, 날짜 및 요일 이름의 렌더링 여부를 확인한다. 날짜 포맷이 테스트 환경의 타임존에 따라 달라질 수 있으므로 날짜 관련 검증에 허용 범위를 둔다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈
- [`../utils/test-utils`](../utils/test-utils.ts.md) — `loadExample`, `getSurface`, `getDeepTextContent`
- [`../../src/local-gallery`](../../src/local-gallery.ts.md) — `LocalGallery` 타입

## Exports

없음 (테스트 파일)

## 테스트 케이스 명세

### `describe('Example: Calendar Day')`

**픽스처 설정:**
- `gallery: LocalGallery`, `surface: HTMLElement`, `textContent: string`을 스위트 범위 변수로 선언.
- `beforeEach`: `loadExample('03_calendar-day.json')`으로 갤러리를 마운트하고, `getSurface(gallery)`로 `a2ui-surface` 엘리먼트를 획득한 뒤, `getDeepTextContent(surface)`를 `textContent`에 저장.
- `afterEach`: `gallery?.remove()`로 DOM에서 갤러리를 제거.

---

**`it('should render text content')`**
- 검증 동작: 일정 항목들의 제목, 시간 범위, 버튼 레이블이 모두 렌더링된다.
- 기대 결과: `textContent`에 `'Lunch'`, `'12:00 - 12:45 PM'`, `'Q1 roadmap review'`, `'1:00 - 2:00 PM'`, `'Team standup'`, `'3:30 - 4:00 PM'`, `'Add to calendar'`, `'Discard'` 포함.

**`it('should render date and day name')`**
- 검증 동작: 입력 날짜 `"2025-12-28"`에 해당하는 날짜 번호와 요일 이름이 렌더링된다.
- 경계 케이스: 테스트 실행 머신의 타임존에 따라 로컬 포맷 시 12월 28일 또는 27일이 표시될 수 있으므로, `'28'` 또는 `'27'` 중 하나가 포함되면 통과한다. `withContext('Should render day number')`로 실패 시 컨텍스트 메시지를 제공한다.
- 요일 이름은 `'Sunday'`, `'Monday'`, `'Tuesday'`, `'Wednesday'`, `'Thursday'`, `'Friday'`, `'Saturday'` 중 하나가 포함되면 통과한다. `withContext('Should render day name')`으로 실패 메시지를 제공한다.

## 동작 흐름

모든 테스트는 `beforeEach`에서 준비된 `textContent`를 사용하는 정적 렌더링 검증이다. 타임존 의존성이 있는 날짜 포맷 검증은 OR 조건(`||`)으로 허용 범위를 넓혀 다양한 실행 환경에서도 안정적으로 동작하도록 설계되었다.
