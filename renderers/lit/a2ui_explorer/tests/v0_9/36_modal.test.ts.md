# renderers/lit/a2ui_explorer/tests/v0_9/36_modal.test.ts

## 개요

`36_modal.json` 예제 파일을 로드하여 Lit 렌더러의 모달(Modal) 컴포넌트가 열기/닫기 상호작용을 올바르게 처리하는지 검증하는 브라우저 통합 테스트 파일이다. 트리거 클릭 → 모달 열림 확인 → 콘텐츠 확인 → 닫기 버튼 클릭 → 모달 닫힘 확인의 전체 흐름을 하나의 테스트 케이스에서 순차적으로 검증한다.

## 의존성

### 저장소 내부 모듈
- [`../utils/test-utils`](../utils/test-utils.ts.md) — `loadExample`, `getSurface`, `getDeepTextContent`, `querySelectorAllDeep`, `whenSettled` 유틸리티 함수
- [`../../src/local-gallery`](../../src/local-gallery.ts.md) — `LocalGallery` 커스텀 엘리먼트 클래스

### 외부 패키지
- 없음 (테스트 프레임워크는 환경에 전역 주입된 `describe`, `it`, `afterEach`, `expect` 사용)

## Exports

없음 (테스트 파일이므로 export 없음)

## 동작 흐름

`describe('Example: Modal', ...)` 블록:

1. `gallery: LocalGallery`와 `surface: HTMLElement` 변수를 블록 스코프에서 선언.
2. **`afterEach`**: `gallery?.remove()`로 DOM 정리.

### 테스트 케이스

**`'should open and close modal'`** (단일 `it` 블록 내 순차 실행):

| 단계 | 검증 동작 | 사용 유틸리티 |
|---|---|---|
| 1. 예제 로드 | `loadExample('36_modal.json')` 비동기 로드 후 `getSurface`로 surface 획득 | `loadExample`, `getSurface` |
| 2. 트리거 존재 확인 | `.a2ui-modal-trigger` CSS 클래스를 가진 첫 번째 엘리먼트가 truthy인지 확인 | `querySelectorAllDeep` |
| 3. 모달 초기 닫힘 확인 | `.a2ui-modal-overlay` 첫 번째 `HTMLDialogElement`의 `open` 속성이 falsy인지 확인 | `querySelectorAllDeep` |
| 4. 트리거 클릭 + 안정화 대기 | `trigger.click()` 후 `whenSettled(gallery)` 대기 | `whenSettled` |
| 5. 모달 열림 확인 | `.a2ui-modal-overlay`의 `open` 속성이 truthy인지 확인 | `querySelectorAllDeep` |
| 6. 모달 콘텐츠 확인 | `.a2ui-modal-overlay` 내 텍스트에 `'This is the content inside the modal.'` 포함 여부 확인 | `getDeepTextContent`, `querySelectorAllDeep` |
| 7. 닫기 버튼 존재 확인 + 클릭 + 안정화 대기 | `.a2ui-modal-close` 첫 번째 엘리먼트가 truthy인지 확인 후 클릭, `whenSettled(gallery)` 대기 | `querySelectorAllDeep`, `whenSettled` |
| 8. 모달 닫힘 확인 | `.a2ui-modal-overlay`의 `open` 속성이 falsy인지 최종 확인 | `querySelectorAllDeep` |
