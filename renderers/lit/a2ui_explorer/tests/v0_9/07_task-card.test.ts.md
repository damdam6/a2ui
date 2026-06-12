# renderers/lit/a2ui_explorer/tests/v0_9/07_task-card.test.ts

## 개요

A2UI 예제 `07_task-card.json`을 `<local-gallery>`에 로드하여 작업 카드 UI가 올바르게 렌더링되는지 검증하는 통합 테스트 파일이다. 텍스트 콘텐츠, `datetime-local` 입력 필드의 날짜 값, 체크박스 존재 여부, 아이콘 존재 여부를 확인한다. 동적 컴포넌트를 위해 `beforeEach`에서 `wait(100)` 추가 대기를 적용한다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈
- [`../utils/test-utils`](../utils/test-utils.ts.md) — `loadExample`, `getSurface`, `getDeepTextContent`, `querySelectorAllDeep`, `whenSettled`
- [`../../src/local-gallery`](../../src/local-gallery.ts.md) — `LocalGallery` 타입

## Exports

없음 (테스트 파일)

## 로컬 유틸리티

### `wait(ms: number): Promise<void>`
`setTimeout(resolve, ms)`를 래핑한 인라인 유틸리티로, 지정된 밀리초 동안 대기한다. `beforeEach`에서 동적 컴포넌트의 렌더링 완료를 보장하기 위해 `wait(100)`으로 호출된다.

## 테스트 케이스 명세

### `describe('Example: Task Card')`

**픽스처 설정:**
- `gallery: LocalGallery`, `surface: HTMLElement`, `textContent: string`을 스위트 범위 변수로 선언.
- `beforeEach`:
  1. `loadExample('07_task-card.json')`으로 갤러리를 마운트하고 `getSurface(gallery)`로 `surface` 획득.
  2. `await wait(100)` — 동적 컴포넌트를 위한 추가 100ms 대기.
  3. `await whenSettled(gallery)` — Lit 업데이트 사이클 완료 대기.
  4. `getDeepTextContent(surface)`를 `textContent`에 저장.
- `afterEach`: `gallery?.remove()`로 DOM에서 갤러리를 제거.

---

**`it('should render text content')`**
- 검증 동작: 태스크 제목, 설명, 마감일 레이블, 태그 이름이 렌더링된다.
- 기대 결과: `textContent`에 `'Review pull request'`, `'Review and approve the authentication module changes.'`, `'Due'`, `'Backend'` 포함.

**`it('should render date time input')`**
- 검증 동작: `type="datetime-local"`인 `<input>` 엘리먼트가 존재하고 날짜 값에 `'2025-12-15'`가 포함된다.
- 구현: `querySelectorAllDeep(surface, 'input')`으로 모든 input을 찾은 후 `find(i => i.type === 'datetime-local')`으로 datetime-local 타입만 필터링. `withContext('Should have a date time input element')`으로 실패 시 컨텍스트 메시지 제공.
- 기대 결과: `dateTimeInput`이 존재하고 `dateTimeInput.value`에 `'2025-12-15'` 포함.

**`it('should render checkbox')`**
- 검증 동작: `type="checkbox"`인 `<input>` 엘리먼트가 존재한다.
- 구현: `querySelectorAllDeep(surface, 'input[type="checkbox"]')[0]`으로 체크박스를 찾음. `withContext('Should have a checkbox')`으로 실패 시 컨텍스트 메시지 제공.
- 기대 결과: 결과 엘리먼트가 존재(truthy).

**`it('should render icon')`**
- 검증 동작: `a2ui-icon` 커스텀 엘리먼트가 존재한다.
- 구현: `querySelectorAllDeep(surface, 'a2ui-icon')[0]`으로 아이콘을 찾음. `withContext('Should have an icon')`으로 실패 시 컨텍스트 메시지 제공.
- 기대 결과: 결과 엘리먼트가 존재(truthy).

## 동작 흐름

이 테스트 스위트는 동적 컴포넌트를 위해 `beforeEach`에 `wait(100)` + `whenSettled` 이중 대기 전략을 사용한다. 텍스트 검증은 준비된 `textContent`로 수행하고, datetime 입력·체크박스·아이콘 검증은 `querySelectorAllDeep`으로 DOM 엘리먼트를 직접 찾아 확인한다.
