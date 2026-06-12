# renderers/lit/a2ui_explorer/tests/v0_9/13_coffee-order.test.ts

## 개요

`13_coffee-order.json` 예제 파일을 로드하여 "Coffee Order" UI의 렌더링 및 버튼 상호작용을 검증하는 브라우저 통합 테스트 파일이다. 주문 항목 텍스트, 아이콘, 그리고 `Purchase`와 `Add to cart` 두 가지 버튼에 대한 클릭 이벤트 처리를 확인한다. `findButtonByText` 유틸리티를 활용하여 텍스트 기반 버튼 탐색을 수행하는 것이 특징이다.

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

`loadExample('13_coffee-order.json')`을 비동기로 호출하여 `gallery`를 초기화한 후 `surface`와 `textContent`를 준비한다.

### afterEach

`gallery?.remove()`로 DOM을 정리한다.

## 테스트 케이스

### `should render text content`

- **검증 동작**: `textContent`에 다음 문자열들이 포함되어 있는지 확인한다: `'Subtotal'`, `'Tax'`, `'Total'`, `'Purchase'`, `'Add to cart'`, `'Sunrise Coffee'`, `'Oat Milk Latte'`, `'Grande, Extra Shot'`, `'Chocolate Croissant'`, `'Warmed'`.
- **픽스처/모킹**: `beforeEach`에서 로드된 `13_coffee-order.json` 예제 데이터.

### `should render icon`

- **검증 동작**: `querySelectorAllDeep(surface, 'a2ui-icon')[0]`로 첫 번째 `a2ui-icon` 커스텀 요소가 truthy인지 확인하고, `textContent`에 아이콘 이름 `'favorite'`가 포함되어 있는지 검증한다.
- **픽스처/모킹**: Shadow DOM 깊이 탐색용 `querySelectorAllDeep` 유틸리티.

### `should handle Purchase button click`

- **검증 동작**: `findButtonByText(surface, 'Purchase')`로 `Purchase` 버튼을 찾아 `click()`을 호출한다. `whenSettled(gallery)` 완료 후 `gallery.actionLog.length`가 `1`이고 `gallery.actionLog[0].name`이 `'purchase'`인지 검증한다.
- **픽스처/모킹**: `findButtonByText`로 텍스트 기반 버튼 탐색, `whenSettled`로 비동기 액션 완료 대기.

### `should handle Add to cart button click`

- **검증 동작**: `findButtonByText(surface, 'Add to cart')`로 `Add to cart` 버튼을 찾아 `click()`을 호출한다. `whenSettled(gallery)` 완료 후 `gallery.actionLog.length`가 `1`이고 `gallery.actionLog[0].name`이 `'add_to_cart'`인지 검증한다.
- **픽스처/모킹**: `findButtonByText`와 `whenSettled` 유틸리티.
