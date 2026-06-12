# renderers/lit/a2ui_explorer/tests/v0_9/29_movie-card.test.ts

## 개요

`29_movie-card.json` 예제를 로드하여 Movie Card 컴포넌트의 렌더링과 모달 트리거 상호작용을 검증하는 통합 테스트 파일이다. 영화 메타데이터 텍스트, 포스터 이미지, 아이콘, 그리고 "Watch Trailer" 버튼 클릭 시 모달이 열리고 비디오 요소가 노출되는지를 확인한다. 모달 정리(cleanup) 로직도 테스트 내에 포함된다.

## 의존성

### 외부 패키지
- (없음)

### 저장소 내부 모듈
- [`../utils/test-utils`](../utils/test-utils.ts.md) — `loadExample`, `getSurface`, `getDeepTextContent`, `querySelectorAllDeep`, `whenSettled` 임포트
- [`../../src/local-gallery`](../../src/local-gallery.ts.md) — `LocalGallery` 클래스 임포트

## Exports

명시적 export 없음.

## 동작 흐름

1. `beforeEach`: `loadExample('29_movie-card.json')` → `getSurface` → `getDeepTextContent`.
2. `afterEach`: `gallery.remove()`.
3. 모달 테스트에서는 `.a2ui-modal-trigger` 버튼 클릭 → `whenSettled` → `.a2ui-modal-overlay`의 `open` 속성 및 내부 `video` 요소 검사 → `.a2ui-modal-close` 버튼 클릭으로 정리.

## 테스트 케이스

### `should render text content`
- **검증 동작**: 텍스트에 `'Interstellar'`, `'(2014)'`, `'Sci-Fi • Adventure • Drama'`, `'8.7/10'`, `'2h 49min'`, `'Watch Trailer'`가 모두 포함되는지 확인한다.
- **픽스처**: `29_movie-card.json`

### `should render image`
- **검증 동작**: 첫 번째 `img`의 `src`가 정확히 `'https://images.unsplash.com/photo-1536440136628-849c177e76a1?w=200&h=300&fit=crop'`인지 확인한다.
- **픽스처**: 동일.

### `should render icons`
- **검증 동작**: 텍스트에 아이콘 이름 `'calendar_today'`와 `'star'`가 포함되는지 확인한다.
- **픽스처**: 동일.

### `should open modal with video when clicking watch trailer button`
- **검증 동작**:
  1. `.a2ui-modal-trigger` 첫 번째 요소가 존재하는지 확인.
  2. 해당 요소를 클릭하고 `whenSettled` 대기.
  3. `.a2ui-modal-overlay` 요소의 `open` 프로퍼티가 truthy인지 확인.
  4. `video` 요소가 존재하고 `src` 속성이 `'https://www.w3schools.com/html/mov_bbb.mp4'`인지 확인.
  5. `.a2ui-modal-close` 버튼을 클릭하고 `whenSettled`로 모달을 닫는 정리 수행.
- **모킹/유틸**: `whenSettled(gallery)`.
