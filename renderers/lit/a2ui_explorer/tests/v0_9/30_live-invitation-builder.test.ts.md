# renderers/lit/a2ui_explorer/tests/v0_9/30_live-invitation-builder.test.ts

## 개요

`30_live-invitation-builder.json` 예제를 로드하여 Live Invitation Builder 컴포넌트의 렌더링과 실시간 미리보기 업데이트 동작을 검증하는 통합 테스트 파일이다. 입력 폼 값 변경 시 Live Preview 카드가 즉시 반영되는지, 그리고 위치 칩(chip) 선택 시 미리보기가 업데이트되는지를 확인한다. 비동기 템플릿 렌더링을 기다리기 위해 파일 내에 `wait`·`waitForCondition` 헬퍼를 직접 정의한다.

## 의존성

### 외부 패키지
- (없음)

### 저장소 내부 모듈
- [`../utils/test-utils`](../utils/test-utils.ts.md) — `loadExample`, `getSurface`, `getDeepTextContent`, `querySelectorAllDeep`, `whenSettled` 임포트
- [`../../src/local-gallery`](../../src/local-gallery.ts.md) — `LocalGallery` 클래스 임포트

## Exports

명시적 export 없음.

## 상세 명세

### `wait(ms: number): Promise<void>`
- 파일 상단에 `const`로 정의된 인라인 헬퍼.
- `setTimeout`을 `Promise`로 래핑하여 `ms` 밀리초 동안 실행을 일시 중단한다.

### `waitForCondition(condition: () => boolean, timeout = 1000, interval = 50): Promise<boolean>`
- 비동기 폴링 헬퍼. `condition` 콜백이 `true`를 반환할 때까지 `interval`ms 간격으로 반복 확인한다.
- `performance.now()`를 기준 시각(`start`)으로 기록하고, `performance.now() - start > timeout`이면 `false`를 반환한다.
- 조건을 만족하면 `true`를 즉시 반환하며, 아직 시간이 남으면 `interval`ms 대기 후 재확인한다.

### `getLivePreview(): HTMLElement` (테스트 스위트 내 헬퍼 함수)
- `querySelectorAllDeep(surface, 'a2ui-card')`로 모든 카드 요소를 조회해 첫 번째 요소를 Live Preview 카드로 간주한다.
- 첫 번째 카드가 없으면 `expect(...).withContext(...).toBeTruthy()` 단언으로 테스트를 실패시킨다.
- 반환 타입은 `HTMLElement`로 캐스팅된다.

## 동작 흐름

1. `beforeEach`: `loadExample('30_live-invitation-builder.json')` → `getSurface` → `getDeepTextContent`.
2. `afterEach`: `gallery.remove()`.
3. 입력 값 변경 테스트: `querySelectorAllDeep(surface, 'input')`으로 전체 `input` 목록을 조회 → `type`이 `'text'` 또는 미지정인 것만 필터링 → `value` 직접 설정 후 `new Event('input')` 디스패치 → `whenSettled` → `getLivePreview()`의 텍스트 재확인.
4. 위치 칩 변경 테스트: `wait(100)` + `whenSettled`로 초기 렌더링 안정화 → `.chip` 요소 조회 → 특정 칩 클릭 → `waitForCondition`으로 미리보기 텍스트 변화 폴링.

## 테스트 케이스

### `should render text content`
- **검증 동작**: `wait(100)` + `whenSettled` 후 텍스트에 `'Invitation Builder'`, `'Customize your invitation'`, `'Live Preview'`가 포함되고, 미리보기 카드 텍스트에 `'Celebrating'`과 `'Location:'`이 포함되는지 확인한다.
- **픽스처**: `30_live-invitation-builder.json`

### `should render date`
- **검증 동작**: 초기 텍스트에 `'July 15'`와 `'2025'`가 포함되는지 확인한다.

### `should render image`
- **검증 동작**: 첫 번째 `img`의 `src`가 `'https://images.unsplash.com/photo-1511795409834-ef04bbd61622?w=400&h=200&fit=crop'`인지 확인한다.

### `should render inputs`
- **검증 동작**: `input` 요소가 2개 이상 존재하는지 확인한다.

### `should update preview when changing Event Name input`
- **검증 동작**: 첫 번째 텍스트 입력(이벤트 이름)에 `'Awesome Party'`, 두 번째 입력(게스트 이름)에 `'Alex Johnson'`을 설정하고 `input` 이벤트 디스패치 → `whenSettled` → Live Preview 카드에 `'Awesome Party'`가 포함되는지 확인한다.

### `should update preview when changing Guest of Honor input`
- **검증 동작**: 이벤트 이름에 `'Summer Gala'`, 게스트 이름에 `'John Doe'` 설정 → `whenSettled` → Preview에 `'John Doe'`가 포함되는지 확인한다.

### `should update preview when changing location`
- **검증 동작**:
  1. `wait(100)` + `whenSettled`로 안정화.
  2. `.chip` 요소 3개 이상 존재 확인.
  3. `'Grand Ballroom'` 칩 클릭 → `waitForCondition`으로 Preview 텍스트에 `'Location: ballroom'` 포함 여부 폴링(최대 1000ms).
  4. `'Sunset Terrace'` 칩 클릭 → `waitForCondition`으로 `'Location: terrace'` 포함 여부 폴링.
- **모킹/유틸**: `wait`, `whenSettled`, `waitForCondition`.
