# renderers/lit/a2ui_explorer/tests/v0_9/17_event-detail.test.ts

## 개요

`17_event-detail.json` 예제 파일을 로드하여 "Event Detail" 이벤트 상세 화면의 렌더링 및 버튼 상호작용을 검증하는 브라우저 통합 테스트 파일이다. 이벤트 제목, 장소, 설명, 날짜, 아이콘, 그리고 `Accept`와 `Decline` 두 가지 RSVP 버튼 클릭에 대한 액션 처리를 포괄적으로 확인한다.

## 의존성

### 외부 패키지
- `jasmine` (전역 테스트 프레임워크)

### 저장소 내부 모듈
- [`../utils/test-utils`](../utils/test-utils.ts.md) — `loadExample`, `getSurface`, `getDeepTextContent`, `querySelectorAllDeep`, `whenSettled`, `findButtonByText` 유틸리티 함수
- [`../../src/local-gallery`](../../src/local-gallery.ts.md) — `LocalGallery` 클래스

## Exports

이 파일은 아무것도 export하지 않는다. Jasmine `describe` 블록으로 테스트 스위트를 등록한다.

## 동작 흐름

### 픽스처 및 상태 변수

- `gallery: LocalGallery` — 각 테스트마다 생성되고 `afterEach`에서 제거된다.
- `surface: HTMLElement` — 렌더링 표면 요소.
- `textContent: string` — Shadow DOM을 포함한 전체 텍스트 콘텐츠.

### beforeEach

`loadExample('17_event-detail.json')`을 비동기로 호출하여 `gallery`를 초기화한 후 `surface`와 `textContent`를 준비한다.

### afterEach

`gallery?.remove()`로 DOM을 정리한다.

## 테스트 케이스

### `should render text content`

- **검증 동작**: `textContent`에 다음이 포함되어 있는지 확인한다. 이벤트명 `'Product Launch Meeting'`, 장소 `'Conference Room A, Building 2'`, 설명 `'Review final product specs and marketing materials before the Q1 launch.'`, 버튼명 `'Accept'`와 `'Decline'`, 날짜 `'Dec 19'`. 날짜는 UTC 환경에 따라 다를 수 있다고 주석에 명시되어 있다.
- **픽스처/모킹**: `beforeEach`에서 로드된 `17_event-detail.json` 예제 데이터.

### `should render icons`

- **검증 동작**: `querySelectorAllDeep(surface, 'a2ui-icon')`로 모든 아이콘 요소를 수집하고 개수가 2 이상인지 확인한다. `textContent`에 아이콘 이름 `'calendar_today'`와 `'location_on'`이 포함되어 있는지 검증한다.
- **픽스처/모킹**: `querySelectorAllDeep` 유틸리티.

### `should handle Accept button click`

- **검증 동작**: `findButtonByText(surface, 'Accept')`로 버튼을 찾아 `click()`을 호출한다. `whenSettled(gallery)` 완료 후 `gallery.actionLog.length`가 `1`이고 `gallery.actionLog[0].name`이 `'accept'`인지 검증한다.
- **픽스처/모킹**: `findButtonByText`와 `whenSettled` 유틸리티.

### `should handle Decline button click`

- **검증 동작**: `findButtonByText(surface, 'Decline')`로 버튼을 찾아 `click()`을 호출한다. `whenSettled(gallery)` 완료 후 `gallery.actionLog.length`가 `1`이고 `gallery.actionLog[0].name`이 `'decline'`인지 검증한다.
- **픽스처/모킹**: `findButtonByText`와 `whenSettled` 유틸리티.
