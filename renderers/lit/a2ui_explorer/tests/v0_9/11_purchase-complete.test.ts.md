# renderers/lit/a2ui_explorer/tests/v0_9/11_purchase-complete.test.ts

## 개요

`11_purchase-complete.json` 예제 파일을 로드하여 "Purchase Complete" UI가 올바르게 렌더링되는지 검증하는 브라우저 통합 테스트 파일이다. 텍스트 콘텐츠, 이미지, 아이콘, 버튼 클릭 동작을 포함한 구매 완료 화면의 전체적인 렌더링 정확성을 확인한다. `LocalGallery` 인스턴스를 통해 예제를 마운트하고, 각 테스트 종료 후 DOM에서 제거하는 생명주기 관리를 수행한다.

## 의존성

### 외부 패키지
- `jasmine` (전역 `describe`, `it`, `beforeEach`, `afterEach`, `expect` — 테스트 프레임워크)

### 저장소 내부 모듈
- [`../utils/test-utils`](../utils/test-utils.ts.md) — `loadExample`, `getSurface`, `getDeepTextContent`, `querySelectorAllDeep`, `whenSettled` 유틸리티 함수
- [`../../src/local-gallery`](../../src/local-gallery.ts.md) — `LocalGallery` 클래스

## Exports

이 파일은 아무것도 export하지 않는다. Jasmine `describe` 블록으로 테스트 스위트를 등록한다.

## 동작 흐름

### 픽스처 및 상태 변수

- `gallery: LocalGallery` — 각 테스트마다 새로 생성되고 `afterEach`에서 `gallery?.remove()`로 DOM에서 제거된다.
- `surface: HTMLElement` — `getSurface(gallery)`로 얻은 렌더링 표면 요소.
- `textContent: string` — `getDeepTextContent(surface)`로 얻은 Shadow DOM을 포함한 전체 텍스트.

### beforeEach

`loadExample('11_purchase-complete.json')`을 비동기로 호출하여 `gallery`를 초기화한다. 이후 `getSurface`로 표면 요소를 추출하고, `getDeepTextContent`로 전체 텍스트 콘텐츠를 수집한다.

### afterEach

`gallery?.remove()`를 호출하여 DOM 정리를 수행한다. `gallery`가 undefined인 경우에도 안전하게 동작하도록 optional chaining을 사용한다.

## 테스트 케이스

### `should render text content`

- **검증 동작**: `textContent`에 다음 문자열이 포함되어 있는지 확인한다: `'Purchase Complete'`, `'Wireless Headphones Pro'`, `'Arrives Dec 18 - Dec 20'`, `'Sold by:'`, `'TechStore Official'`, `'View Order Details'`, `'199.99'`(통화 포맷된 가격).
- **픽스처/모킹**: `beforeEach`에서 로드된 `11_purchase-complete.json` 예제 데이터.

### `should render image`

- **검증 동작**: `querySelectorAllDeep(surface, 'img')[0]`으로 첫 번째 `<img>` 요소를 선택하고, truthy 여부 및 `src` 속성이 `'https://images.unsplash.com/photo-1505740420928-5e560c06d30e?w=100&h=100&fit=crop'`과 일치하는지 확인한다.
- **픽스처/모킹**: Shadow DOM을 포함한 `querySelectorAllDeep` 유틸리티.

### `should render icons`

- **검증 동작**: `textContent`에 아이콘 이름 `'check'`와 `'arrow_forward'`가 포함되어 있는지 확인한다.
- **픽스처/모킹**: 없음.

### `should handle button click`

- **검증 동작**: `querySelectorAllDeep(surface, '.a2ui-button')[0]`으로 버튼을 선택하고, 해당 요소가 truthy임을 `.withContext('Should find View Order Details button')`으로 맥락 정보와 함께 확인한다. 버튼을 `click()`한 뒤 `whenSettled(gallery)`를 `await`하여 비동기 처리가 완료되기를 기다린다. 이후 `gallery.actionLog.length`가 `1`이고 `gallery.actionLog[0].name`이 `'view_details'`인지 검증한다.
- **픽스처/모킹**: `whenSettled` 유틸리티로 액션 처리 완료를 대기.
