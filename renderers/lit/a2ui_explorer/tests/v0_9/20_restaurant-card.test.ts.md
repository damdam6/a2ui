# renderers/lit/a2ui_explorer/tests/v0_9/20_restaurant-card.test.ts

## 개요

`20_restaurant-card.json` 예제 파일을 로드하여 "Restaurant Card" 레스토랑 카드 UI의 정적 렌더링을 검증하는 브라우저 통합 테스트 파일이다. 레스토랑 이름, 가격대, 카테고리, 평점, 리뷰 수, 거리, 배달 예상 시간, 이미지 및 아이콘의 정확한 표시를 확인한다. 버튼 클릭 상호작용은 없다.

## 의존성

### 외부 패키지
- `jasmine` (전역 테스트 프레임워크)

### 저장소 내부 모듈
- [`../utils/test-utils`](../utils/test-utils.ts.md) — `loadExample`, `getSurface`, `getDeepTextContent`, `querySelectorAllDeep` 유틸리티 함수
- [`../../src/local-gallery`](../../src/local-gallery.ts.md) — `LocalGallery` 클래스

## Exports

이 파일은 아무것도 export하지 않는다. Jasmine `describe` 블록으로 테스트 스위트를 등록한다.

## 동작 흐름

### 픽스처 및 상태 변수

- `gallery: LocalGallery` — 각 테스트마다 생성되고 `afterEach`에서 제거된다.
- `surface: HTMLElement` — 렌더링 표면 요소.
- `textContent: string` — Shadow DOM을 포함한 전체 텍스트 콘텐츠.

### beforeEach

`loadExample('20_restaurant-card.json')`을 비동기로 호출하여 `gallery`를 초기화한 후 `surface`와 `textContent`를 준비한다.

### afterEach

`gallery?.remove()`로 DOM을 정리한다.

## 테스트 케이스

### `should render text content`

- **검증 동작**: `textContent`에 다음 문자열들이 포함되어 있는지 확인한다: 식당명 `'The Italian Kitchen'`, 가격대 `'$$$'`, 카테고리 `'Italian'`, 평점 `'4.8'`, 리뷰 수 `'2,847 reviews'`, 거리 `'0.8 mi'`, 배달 예상 시간 `'25-35 min'`.
- **픽스처/모킹**: `beforeEach`에서 로드된 `20_restaurant-card.json` 예제 데이터.

### `should render image`

- **검증 동작**: `querySelectorAllDeep(surface, 'img')[0]`으로 첫 번째 `<img>` 요소를 선택하고, truthy 여부 및 `src` 속성이 `'https://images.unsplash.com/photo-1517248135467-4c7edcad34c4?w=300&h=150&fit=crop'`과 일치하는지 확인한다.
- **픽스처/모킹**: `querySelectorAllDeep` 유틸리티.

### `should render icon`

- **검증 동작**: `querySelectorAllDeep(surface, 'a2ui-icon')[0]`이 truthy인지 확인한다. 아이콘 이름 검증은 수행하지 않는다.
- **픽스처/모킹**: `querySelectorAllDeep` 유틸리티.
