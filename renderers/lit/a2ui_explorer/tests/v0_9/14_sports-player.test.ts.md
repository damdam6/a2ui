# renderers/lit/a2ui_explorer/tests/v0_9/14_sports-player.test.ts

## 개요

`14_sports-player.json` 예제 파일을 로드하여 "Sports Player" 선수 프로필 카드 UI의 렌더링을 검증하는 브라우저 통합 테스트 파일이다. 선수 이름, 등번호, 소속 팀, 경기 스탯(PPG, RPG, APG), 프로필 이미지의 정확한 렌더링을 확인한다. 버튼 상호작용은 없으며 정적 콘텐츠 렌더링에만 집중한다.

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

`loadExample('14_sports-player.json')`을 비동기로 호출하여 `gallery`를 초기화한 후 `surface`와 `textContent`를 준비한다.

### afterEach

`gallery?.remove()`로 DOM을 정리한다.

## 테스트 케이스

### `should render text content`

- **검증 동작**: `textContent`에 다음 문자열들이 포함되어 있는지 확인한다: 선수명 `'Marcus Johnson'`, 등번호 `'#23'`, 팀명 `'LA Lakers'`, 득점 `'28.4'`와 라벨 `'PPG'`, 리바운드 `'7.2'`와 라벨 `'RPG'`, 어시스트 `'6.8'`와 라벨 `'APG'`.
- **픽스처/모킹**: `beforeEach`에서 로드된 `14_sports-player.json` 예제 데이터.

### `should render image`

- **검증 동작**: `querySelectorAllDeep(surface, 'img')[0]`으로 첫 번째 `<img>` 요소를 선택하고, truthy 여부 및 `src` 속성이 `'https://images.unsplash.com/photo-1546519638-68e109498ffc?w=200&h=200&fit=crop'`과 일치하는지 확인한다.
- **픽스처/모킹**: Shadow DOM을 포함한 `querySelectorAllDeep` 유틸리티.
