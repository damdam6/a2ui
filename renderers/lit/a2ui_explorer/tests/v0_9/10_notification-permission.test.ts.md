# renderers/lit/a2ui_explorer/tests/v0_9/10_notification-permission.test.ts

## 개요

A2UI 예제 `10_notification-permission.json`을 `<local-gallery>`에 로드하여 알림 권한 요청 UI가 올바르게 렌더링되고, "Yes"/"No" 버튼 클릭 시 `gallery.actionLog`에 올바른 액션이 기록되는지 검증하는 통합 테스트 파일이다. 텍스트 콘텐츠, 아이콘 내용, 버튼 인터랙션 결과를 모두 검증한다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈
- [`../utils/test-utils`](../utils/test-utils.ts.md) — `loadExample`, `getSurface`, `getDeepTextContent`, `querySelectorAllDeep`, `whenSettled`, `findButtonByText`
- [`../../src/local-gallery`](../../src/local-gallery.ts.md) — `LocalGallery` 타입

## Exports

없음 (테스트 파일)

## 테스트 케이스 명세

### `describe('Example: Notification Permission')`

**픽스처 설정:**
- `gallery: LocalGallery`, `surface: HTMLElement`, `textContent: string`을 스위트 범위 변수로 선언.
- `beforeEach`: `loadExample('10_notification-permission.json')`으로 갤러리를 마운트하고, `getSurface(gallery)`로 `a2ui-surface` 엘리먼트를 획득한 뒤, `getDeepTextContent(surface)`를 `textContent`에 저장.
- `afterEach`: `gallery?.remove()`로 DOM에서 갤러리를 제거.

---

**`it('should render text content')`**
- 검증 동작: 알림 권한 요청 카드의 제목, 설명, 버튼 레이블이 렌더링된다.
- 기대 결과: `textContent`에 `'Enable notification'`, `'Get alerts for order status changes'`, `'Yes'`, `'No'` 포함.

**`it('should render icon')`**
- 검증 동작: `a2ui-icon` 커스텀 엘리먼트가 존재하고 텍스트 콘텐츠로 `'check'`를 포함한다.
- 구현: `querySelectorAllDeep(surface, 'a2ui-icon')[0]`으로 아이콘 엘리먼트를 찾은 뒤 `getDeepTextContent(iconEl)`이 `'check'`를 포함하는지 확인.
- 기대 결과: 아이콘 엘리먼트가 존재(truthy)하고 텍스트에 `'check'` 포함.

**`it('should handle Yes button click')`**
- 검증 동작: "Yes" 텍스트를 포함하는 버튼을 클릭하면 `gallery.actionLog`에 `name: 'accept'`인 액션이 1건 추가된다.
- 구현: `findButtonByText(surface, 'Yes')`로 버튼을 찾아 `yesBtn.click()`을 호출한 후 `whenSettled(gallery)`를 await.
- 기대 결과: `gallery.actionLog.length === 1`, `gallery.actionLog[0].name === 'accept'`.

**`it('should handle No button click')`**
- 검증 동작: "No" 텍스트를 포함하는 버튼을 클릭하면 `gallery.actionLog`에 `name: 'decline'`인 액션이 1건 추가된다.
- 구현: `findButtonByText(surface, 'No')`로 버튼을 찾아 `noBtn.click()`을 호출한 후 `whenSettled(gallery)`를 await.
- 기대 결과: `gallery.actionLog.length === 1`, `gallery.actionLog[0].name === 'decline'`.

## 동작 흐름

정적 렌더링 검증 테스트들은 `beforeEach`에서 준비된 `textContent`와 `surface`를 사용한다. 아이콘 테스트는 `querySelectorAllDeep`으로 DOM 엘리먼트를 찾아 텍스트 내용을 검증한다. 버튼 클릭 테스트들은 `findButtonByText`로 버튼을 찾고, `click()` 후 `whenSettled`로 업데이트 완료를 기다린 뒤 `actionLog`의 액션 이름을 검증한다. `beforeEach`마다 `loadExample`이 새로 호출되어 각 테스트의 `actionLog`는 독립적이다.
