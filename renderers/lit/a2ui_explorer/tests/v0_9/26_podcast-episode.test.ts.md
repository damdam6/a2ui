# renderers/lit/a2ui_explorer/tests/v0_9/26_podcast-episode.test.ts

## 개요

`26_podcast-episode.json` 예제를 로드하여 Podcast Episode 컴포넌트의 렌더링을 검증하는 통합 테스트 파일이다. 에피소드 메타데이터(제목·시리즈·길이·연도·설명), 커버 이미지, 오디오 플레이어 요소의 존재와 소스 URL을 확인한다.

## 의존성

### 외부 패키지
- (없음)

### 저장소 내부 모듈
- [`../utils/test-utils`](../utils/test-utils.ts.md) — `loadExample`, `getSurface`, `getDeepTextContent`, `querySelectorAllDeep` 임포트
- [`../../src/local-gallery`](../../src/local-gallery.ts.md) — `LocalGallery` 클래스 임포트

## Exports

명시적 export 없음.

## 동작 흐름

1. `beforeEach`: `loadExample('26_podcast-episode.json')` → `getSurface` → `getDeepTextContent`.
2. `afterEach`: `gallery.remove()`.
3. 미디어 요소 검사 시 `querySelectorAllDeep`로 Shadow DOM 내부의 `img` 및 `audio` 태그를 직접 조회하고 속성 값을 단언한다.

## 테스트 케이스

### `should render text content`
- **검증 동작**: 텍스트에 `'Tech Talk Daily'`, `'The Future of AI in Product Design'`, `'45 min'`, `'2024'`, `'How AI is transforming the way we design and build products.'`가 모두 포함되는지 확인한다.
- **픽스처**: `26_podcast-episode.json`

### `should render image`
- **검증 동작**: 첫 번째 `img` 요소가 존재하고 `src` 속성이 정확히 `'https://images.unsplash.com/photo-1478737270239-2f02b77fc618?w=100&h=100&fit=crop'`인지 확인한다.
- **픽스처**: 동일.

### `should render audio player`
- **검증 동작**: `audio` 요소가 존재하고 `src` 속성이 정확히 `'https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3'`인지 확인한다.
- **픽스처**: 동일.
