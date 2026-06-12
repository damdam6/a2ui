# renderers/lit/a2ui_explorer/tests/v0_9/25_contact-card.test.ts

## 개요

`25_contact-card.json` 예제를 로드하여 Contact Card 컴포넌트의 렌더링과 버튼 액션 동작을 검증하는 통합 테스트 파일이다. 연락처 정보(이름·직함·전화번호·이메일·위치), 프로필 이미지 URL, 아이콘 종류, Call/Message 버튼 클릭 후 `gallery.actionLog` 기록을 확인한다.

## 의존성

### 외부 패키지
- (없음)

### 저장소 내부 모듈
- [`../utils/test-utils`](../utils/test-utils.ts.md) — `loadExample`, `getSurface`, `getDeepTextContent`, `querySelectorAllDeep`, `whenSettled`, `findButtonByText` 임포트
- [`../../src/local-gallery`](../../src/local-gallery.ts.md) — `LocalGallery` 클래스 임포트

## Exports

명시적 export 없음.

## 동작 흐름

1. `beforeEach`: `loadExample('25_contact-card.json')` → `getSurface` → `getDeepTextContent`.
2. `afterEach`: `gallery.remove()`.
3. 버튼 액션 테스트에서는 `findButtonByText(surface, '버튼레이블')`로 버튼 요소를 취득하고, `.click()` 후 `whenSettled`를 기다린 뒤 `gallery.actionLog`를 검사한다.

## 테스트 케이스

### `should render text content`
- **검증 동작**: 텍스트에 `'David Park'`, `'Engineering Manager'`, `'+1 (555) 234-5678'`, `'david.park@company.com'`, `'San Francisco, CA'`, `'Call'`, `'Message'`가 모두 포함되는지 확인한다.
- **픽스처**: `25_contact-card.json`

### `should render image`
- **검증 동작**: Shadow DOM을 탐색해 첫 번째 `img` 요소의 `src` 속성이 정확히 `'https://images.unsplash.com/photo-1507003211169-0a1dd7228f2d?w=100&h=100&fit=crop'`인지 확인한다.
- **픽스처**: 동일.

### `should render icons`
- **검증 동작**: `a2ui-icon` 요소가 3개 이상 존재하고, 텍스트에 `'location_on'`, `'phone'`, `'mail'` 아이콘 이름이 포함되는지 확인한다.
- **픽스처**: 동일.

### `should handle Call button click`
- **검증 동작**: `findButtonByText(surface, 'Call')`로 버튼을 취득 → 클릭 → `whenSettled` → `gallery.actionLog.length`가 1이고 `actionLog[0].name`이 `'call'`인지 확인한다.
- **모킹/유틸**: `findButtonByText`, `whenSettled`.

### `should handle Message button click`
- **검증 동작**: `findButtonByText(surface, 'Message')`로 버튼 취득 → 클릭 → `whenSettled` → `gallery.actionLog.length`가 1이고 `actionLog[0].name`이 `'message'`인지 확인한다.
- **모킹/유틸**: `findButtonByText`, `whenSettled`.
