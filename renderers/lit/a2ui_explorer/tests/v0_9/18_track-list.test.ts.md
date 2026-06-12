# renderers/lit/a2ui_explorer/tests/v0_9/18_track-list.test.ts

## 개요

`18_track-list.json` 예제 파일을 로드하여 "Track List" 음악 트랙 목록 화면의 렌더링을 검증하는 브라우저 통합 테스트 파일이다. 플레이리스트명, 3개 트랙 각각의 제목·아티스트·재생 시간, 그리고 앨범 아트 이미지 URL을 개별 테스트로 분리하여 확인한다. 버튼 상호작용은 없다.

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

`loadExample('18_track-list.json')`을 비동기로 호출하여 `gallery`를 초기화한 후 `surface`와 `textContent`를 준비한다.

### afterEach

`gallery?.remove()`로 DOM을 정리한다.

## 테스트 케이스

### `should render playlist name`

- **검증 동작**: `textContent`에 플레이리스트명 `'Focus Flow'`가 포함되어 있는지 확인한다.
- **픽스처/모킹**: `beforeEach`에서 로드된 `18_track-list.json` 예제 데이터.

### `should render Weightless track details`

- **검증 동작**: `textContent`에 트랙명 `'Weightless'`, 아티스트명 `'Marconi Union'`, 재생 시간 `'8:09'`가 포함되어 있는지 확인한다.
- **픽스처/모킹**: 없음.

### `should render Clair de Lune track details`

- **검증 동작**: `textContent`에 트랙명 `'Clair de Lune'`, 아티스트명 `'Debussy'`, 재생 시간 `'5:12'`가 포함되어 있는지 확인한다.
- **픽스처/모킹**: 없음.

### `should render Ambient Light track details`

- **검증 동작**: `textContent`에 트랙명 `'Ambient Light'`, 아티스트명 `'Brian Eno'`, 재생 시간 `'6:45'`가 포함되어 있는지 확인한다.
- **픽스처/모킹**: 없음.

### `should render images`

- **검증 동작**: `querySelectorAllDeep(surface, 'img')`로 모든 이미지를 수집하고 개수가 3 이상인지 확인한다. `src` 속성 배열에 다음 3개의 앨범 아트 URL이 모두 포함되어 있는지 검증한다: `'https://images.unsplash.com/photo-1470225620780-dba8ba36b745?w=50&h=50&fit=crop'`, `'https://images.unsplash.com/photo-1511379938547-c1f69419868d?w=50&h=50&fit=crop'`, `'https://images.unsplash.com/photo-1507838153414-b4b713384a76?w=50&h=50&fit=crop'`. NodeList를 `[...imgs]`로 배열 변환 후 `HTMLImageElement`로 캐스팅하여 `map`으로 src 배열을 생성한다.
- **픽스처/모킹**: `querySelectorAllDeep` 유틸리티.
