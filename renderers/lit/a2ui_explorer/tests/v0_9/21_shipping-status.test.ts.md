# renderers/lit/a2ui_explorer/tests/v0_9/21_shipping-status.test.ts

## 개요

`21_shipping-status.json` 예제 파일을 로드하여 "Shipping Status" 배송 상태 화면의 정적 렌더링을 검증하는 브라우저 통합 테스트 파일이다. 패키지 상태 제목, 운송장 번호, 배송 단계 텍스트(Order Placed → Shipped → Out for Delivery → Delivered), 예상 배달 일시, 그리고 각 단계별 아이콘을 확인한다. 이 파일은 `querySelectorAllDeep` 없이 `getDeepTextContent`만으로 아이콘 존재 여부를 확인하는 가장 단순한 구조다.

## 의존성

### 외부 패키지
- `jasmine` (전역 테스트 프레임워크)

### 저장소 내부 모듈
- [`../utils/test-utils`](../utils/test-utils.ts.md) — `loadExample`, `getSurface`, `getDeepTextContent` 유틸리티 함수 (`querySelectorAllDeep`, `whenSettled`, `findButtonByText` 미사용)
- [`../../src/local-gallery`](../../src/local-gallery.ts.md) — `LocalGallery` 클래스

## Exports

이 파일은 아무것도 export하지 않는다. Jasmine `describe` 블록으로 테스트 스위트를 등록한다.

## 동작 흐름

### 픽스처 및 상태 변수

- `gallery: LocalGallery` — 각 테스트마다 생성되고 `afterEach`에서 제거된다.
- `surface: HTMLElement` — 렌더링 표면 요소.
- `textContent: string` — Shadow DOM을 포함한 전체 텍스트 콘텐츠.

### beforeEach

`loadExample('21_shipping-status.json')`을 비동기로 호출하여 `gallery`를 초기화한 후 `surface`와 `textContent`를 준비한다.

### afterEach

`gallery?.remove()`로 DOM을 정리한다.

## 테스트 케이스

### `should render text content`

- **검증 동작**: `textContent`에 다음 문자열들이 포함되어 있는지 확인한다: 헤더 `'Package Status'`, 운송장 번호가 포함된 `'Tracking: 1Z999AA10123456784'`, 배송 단계 `'Order Placed'`·`'Shipped'`·`'Out for Delivery'`·`'Delivered'`, 배달 예정 시간 `'Estimated delivery: Today by 8 PM'`.
- **픽스처/모킹**: `beforeEach`에서 로드된 `21_shipping-status.json` 예제 데이터.

### `should render icons`

- **검증 동작**: `textContent`에 아이콘 이름 `'info'`(현재 진행 상태), `'check'`(완료 단계), `'send'`(배송 중), `'calendar_today'`(주문 접수)가 포함되어 있는지 확인한다. DOM 쿼리 없이 텍스트 콘텐츠만으로 아이콘 존재를 검증한다.
- **픽스처/모킹**: 없음.
