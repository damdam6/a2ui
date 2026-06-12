# renderers/lit/a2ui_explorer/tests/v0_9/12_chat-message.test.ts

## 개요

`12_chat-message.json` 예제 파일을 로드하여 "Chat Message" UI가 올바르게 렌더링되는지 검증하는 브라우저 통합 테스트 파일이다. 채널명, 사용자 메시지 텍스트, 프로필 이미지 URL을 포함한 채팅 메시지 화면의 렌더링 정확성을 확인한다. 이 테스트 파일에는 버튼 상호작용 테스트가 없으며 정적인 렌더링 검증에 집중한다.

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

`loadExample('12_chat-message.json')`을 비동기로 호출하여 `gallery`를 초기화하고, `getSurface`와 `getDeepTextContent`로 검증에 필요한 상태를 준비한다.

### afterEach

`gallery?.remove()`로 DOM을 정리한다.

## 테스트 케이스

### `should render channel name`

- **검증 동작**: `textContent`에 채널명 `'project-updates'`가 포함되어 있는지 확인한다.
- **픽스처/모킹**: `beforeEach`에서 로드된 `12_chat-message.json` 예제 데이터.

### `should render messages content`

- **검증 동작**: `textContent`에 첫 번째 발신자 이름 `'Mike Chen'`과 메시지 본문 `'Just pushed the new API changes. Ready for review.'`, 두 번째 발신자 이름 `'Sarah Kim'`과 메시지 본문 `"Great! I'll take a look after standup."`이 모두 포함되어 있는지 확인한다.
- **픽스처/모킹**: 없음.

### `should render images`

- **검증 동작**: `querySelectorAllDeep(surface, 'img')`로 모든 `<img>` 요소를 수집하고, 총 개수가 2 이상인지 확인한다. 각 이미지의 `src` 속성 배열에 Mike Chen의 프로필 이미지 URL인 `'https://images.unsplash.com/photo-1472099645785-5658abf4ff4e?w=40&h=40&fit=crop'`와 Sarah Kim의 프로필 이미지 URL인 `'https://images.unsplash.com/photo-1438761681033-6461ffad8d80?w=40&h=40&fit=crop'`가 포함되어 있는지 검증한다. `[...imgs]`로 NodeList를 배열로 변환한 후 `HTMLImageElement`로 타입 캐스팅하여 `map`을 사용한다.
- **픽스처/모킹**: Shadow DOM을 포함한 `querySelectorAllDeep` 유틸리티.
