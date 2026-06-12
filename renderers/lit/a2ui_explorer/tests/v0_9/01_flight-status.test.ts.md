# renderers/lit/a2ui_explorer/tests/v0_9/01_flight-status.test.ts

## 개요

A2UI 예제 `01_flight-status.json`을 `<local-gallery>`에 로드하여 항공편 상태 카드 UI가 올바르게 렌더링되는지 검증하는 통합 테스트 파일이다. 텍스트 콘텐츠(항공편 번호, 출발/도착지, 레이블, 상태), 그리고 아이콘 존재 여부를 확인한다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈
- [`../utils/test-utils`](../utils/test-utils.ts.md) — `loadExample`, `getSurface`, `getDeepTextContent`, `querySelectorAllDeep`
- [`../../src/local-gallery`](../../src/local-gallery.ts.md) — `LocalGallery` 타입

## Exports

없음 (테스트 파일)

## 테스트 케이스 명세

### `describe('Example: Flight Status')`

**픽스처 설정:**
- `gallery: LocalGallery`, `surface: HTMLElement`, `textContent: string`을 스위트 범위 변수로 선언.
- `beforeEach`: `loadExample('01_flight-status.json')`으로 갤러리를 마운트하고, `getSurface(gallery)`로 `a2ui-surface` 엘리먼트를 획득한 뒤, `getDeepTextContent(surface)`로 전체 텍스트를 추출하여 `textContent`에 저장.
- `afterEach`: `gallery?.remove()`로 DOM에서 갤러리를 제거.

---

**`it('should render flight details')`**
- 검증 동작: 항공편의 핵심 식별 정보가 렌더링된다.
- 기대 결과: `textContent`에 `'OS 87'`, `'Vienna'`, `'→'`, `'New York'` 포함.

**`it('should render labels')`**
- 검증 동작: 항공편 상세 정보의 레이블과 상태 문자열이 렌더링된다.
- 기대 결과: `textContent`에 `'Departs'`, `'Arrives'`, `'Status'`, `'On Time'` 포함.

**`it('should render icon')`**
- 검증 동작: `a2ui-icon` 커스텀 엘리먼트가 존재하고 텍스트 콘텐츠로 `'send'`를 렌더링한다.
- 구현: `querySelectorAllDeep(surface, 'a2ui-icon')[0]`으로 아이콘 엘리먼트를 찾은 뒤, `getDeepTextContent(iconInnerEl).trim()`이 `'send'`와 정확히 일치하는지 확인.
- 기대 결과: 아이콘 엘리먼트가 존재(truthy)하고 텍스트가 `'send'`.

## 동작 흐름

각 `it` 블록은 `beforeEach`에서 이미 준비된 `textContent`와 `surface`를 사용한다. 아이콘 테스트는 추가로 `querySelectorAllDeep`을 호출한다. 인터랙션(버튼 클릭 등)은 없으며 렌더링 결과의 정적 검증만 수행한다.
