# renderers/lit/a2ui_explorer/tests/v0_9/19_software-purchase.test.ts

## 개요

`19_software-purchase.json` 예제 파일을 로드하여 "Software Purchase" 소프트웨어 라이선스 구매 화면의 렌더링 및 버튼 상호작용을 검증하는 브라우저 통합 테스트 파일이다. 제품명, 좌석 수, 결제 주기, 총액 표시와 함께 `Confirm Purchase`와 `Cancel` 버튼의 클릭 처리를 확인한다. 이 파일은 `querySelectorAllDeep`을 import하지 않는 점이 다른 파일들과 구별된다.

## 의존성

### 외부 패키지
- `jasmine` (전역 테스트 프레임워크)

### 저장소 내부 모듈
- [`../utils/test-utils`](../utils/test-utils.ts.md) — `loadExample`, `getSurface`, `getDeepTextContent`, `whenSettled`, `findButtonByText` 유틸리티 함수 (`querySelectorAllDeep`은 미사용)
- [`../../src/local-gallery`](../../src/local-gallery.ts.md) — `LocalGallery` 클래스

## Exports

이 파일은 아무것도 export하지 않는다. Jasmine `describe` 블록으로 테스트 스위트를 등록한다.

## 동작 흐름

### 픽스처 및 상태 변수

- `gallery: LocalGallery` — 각 테스트마다 생성되고 `afterEach`에서 제거된다.
- `surface: HTMLElement` — 렌더링 표면 요소.
- `textContent: string` — Shadow DOM을 포함한 전체 텍스트 콘텐츠.

### beforeEach

`loadExample('19_software-purchase.json')`을 비동기로 호출하여 `gallery`를 초기화한 후 `surface`와 `textContent`를 준비한다.

### afterEach

`gallery?.remove()`로 DOM을 정리한다.

## 테스트 케이스

### `should render text content`

- **검증 동작**: `textContent`에 다음 문자열들이 포함되어 있는지 확인한다: `'Purchase License'`, `'Design Suite Pro'`, `'Number of seats'`, `'10 seats'`, `'Billing period'`, `'Annual'`, `'Monthly'`, `'Total'`, `'Confirm Purchase'`, `'Cancel'`.
- **픽스처/모킹**: `beforeEach`에서 로드된 `19_software-purchase.json` 예제 데이터.

### `should render formatted currency`

- **검증 동작**: `textContent`에 포맷된 통화 값 `'1,188'`이 포함되어 있는지 확인한다.
- **픽스처/모킹**: 없음.

### `should handle Confirm Purchase button click`

- **검증 동작**: `findButtonByText(surface, 'Confirm Purchase')`로 버튼을 찾아 `click()`을 호출한다. `whenSettled(gallery)` 완료 후 `gallery.actionLog.length`가 `1`이고 `gallery.actionLog[0].name`이 `'confirm'`인지 검증한다.
- **픽스처/모킹**: `findButtonByText`와 `whenSettled` 유틸리티.

### `should handle Cancel button click`

- **검증 동작**: `findButtonByText(surface, 'Cancel')`로 버튼을 찾아 `click()`을 호출한다. `whenSettled(gallery)` 완료 후 `gallery.actionLog.length`가 `1`이고 `gallery.actionLog[0].name`이 `'cancel'`인지 검증한다.
- **픽스처/모킹**: `findButtonByText`와 `whenSettled` 유틸리티.
