# renderers/lit/a2ui_explorer/tests/v0_9/02_email-compose.test.ts

## 개요

A2UI 예제 `02_email-compose.json`을 `<local-gallery>`에 로드하여 이메일 작성 UI가 올바르게 렌더링되고, 버튼 클릭 시 `gallery.actionLog`에 올바른 액션이 기록되는지 검증하는 통합 테스트 파일이다. 렌더링 검증과 인터랙션 검증을 모두 수행한다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈
- [`../utils/test-utils`](../utils/test-utils.ts.md) — `loadExample`, `getSurface`, `getDeepTextContent`, `whenSettled`, `findButtonByText`
- [`../../src/local-gallery`](../../src/local-gallery.ts.md) — `LocalGallery` 타입

## Exports

없음 (테스트 파일)

## 테스트 케이스 명세

### `describe('Example: Email Compose')`

**픽스처 설정:**
- `gallery: LocalGallery`, `surface: HTMLElement`을 스위트 범위 변수로 선언.
- `beforeEach`: `loadExample('02_email-compose.json')`으로 갤러리를 마운트하고, `getSurface(gallery)`로 `a2ui-surface` 엘리먼트를 획득.
- `afterEach`: `gallery.remove()`로 DOM에서 갤러리를 제거.

---

**`it('should render text content')`**
- 검증 동작: 이메일 작성 폼의 필드 레이블, 버튼 텍스트, 주소 필드 값이 모두 렌더링된다.
- 구현: `getDeepTextContent(surface)`를 호출하여 `textContent`를 구함.
- 기대 결과: `textContent`에 `'FROM'`, `'TO'`, `'SUBJECT'`, `'Send email'`, `'Discard'`, `'alex@acme.com'` 포함.

**`it('should handle Send button click')`**
- 검증 동작: "Send" 텍스트를 포함하는 버튼을 클릭하면 `gallery.actionLog`에 `name: 'send'`인 액션이 1건 추가된다.
- 구현: `findButtonByText(surface, 'Send')`로 버튼을 찾아 `sendBtn.click()`을 호출한 후 `whenSettled(gallery)`를 await.
- 기대 결과: `gallery.actionLog.length === 1`, `gallery.actionLog[0].name === 'send'`.

**`it('should handle Discard button click')`**
- 검증 동작: "Discard" 텍스트를 포함하는 버튼을 클릭하면 `gallery.actionLog`에 `name: 'discard'`인 액션이 1건 추가된다.
- 구현: `findButtonByText(surface, 'Discard')`로 버튼을 찾아 `discardBtn.click()`을 호출한 후 `whenSettled(gallery)`를 await.
- 기대 결과: `gallery.actionLog.length === 1`, `gallery.actionLog[0].name === 'discard'`.

## 동작 흐름

렌더링 검증 테스트는 `beforeEach`에서 준비된 `surface`를 사용하여 정적 콘텐츠를 확인한다. 버튼 클릭 테스트들은 `findButtonByText`로 버튼을 찾고 `click()`을 호출한 뒤 `whenSettled`로 Lit 업데이트 사이클이 완료될 때까지 대기하고, `gallery.actionLog`를 통해 액션 기록을 검증한다. `beforeEach`마다 `loadExample`을 새로 호출하므로 각 테스트의 `actionLog`는 독립적으로 초기화된다.
