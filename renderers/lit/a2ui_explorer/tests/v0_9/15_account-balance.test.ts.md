# renderers/lit/a2ui_explorer/tests/v0_9/15_account-balance.test.ts

## 개요

`15_account-balance.json` 예제 파일을 로드하여 "Account Balance" 계좌 잔액 화면의 렌더링 및 버튼 상호작용을 검증하는 브라우저 통합 테스트 파일이다. 계좌명, 잔액 텍스트, 아이콘, 그리고 `Transfer`와 `Pay Bill` 두 개의 버튼 클릭에 대한 액션 처리를 확인한다.

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

`loadExample('15_account-balance.json')`을 비동기로 호출하여 `gallery`를 초기화한 후 `surface`와 `textContent`를 준비한다.

### afterEach

`gallery?.remove()`로 DOM을 정리한다.

## 테스트 케이스

### `should render text content`

- **검증 동작**: `textContent`에 계좌명 `'Primary Checking'`, 업데이트 시간 `'Updated just now'`, 버튼명 `'Transfer'`와 `'Pay Bill'`, 잔액 `'12,458.32'`(포맷된 통화 값)가 포함되어 있는지 확인한다. 잔액 검증은 "best effort"로 주석 처리되어 있다.
- **픽스처/모킹**: `beforeEach`에서 로드된 `15_account-balance.json` 예제 데이터.

### `should render icon`

- **검증 동작**: `querySelectorAllDeep(surface, 'a2ui-icon')[0]`이 truthy인지 확인하고, `textContent`에 아이콘 이름 `'payment'`가 포함되어 있는지 검증한다.
- **픽스처/모킹**: `querySelectorAllDeep` 유틸리티.

### `should handle Transfer button click`

- **검증 동작**: `findButtonByText(surface, 'Transfer')`로 버튼을 찾아 `click()`을 호출한다. `whenSettled(gallery)` 완료 후 `gallery.actionLog.length`가 `1`이고 `gallery.actionLog[0].name`이 `'transfer'`인지 검증한다.
- **픽스처/모킹**: `findButtonByText`와 `whenSettled` 유틸리티.

### `should handle Pay Bill button click`

- **검증 동작**: `findButtonByText(surface, 'Pay Bill')`로 버튼을 찾아 `click()`을 호출한다. `whenSettled(gallery)` 완료 후 `gallery.actionLog.length`가 `1`이고 `gallery.actionLog[0].name`이 `'pay_bill'`인지 검증한다.
- **픽스처/모킹**: `findButtonByText`와 `whenSettled` 유틸리티.
