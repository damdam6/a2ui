# renderers/lit/a2ui_explorer/tests/v0_9/24_recipe-card.test.ts

## 개요

`24_recipe-card.json` 예제를 로드하여 Recipe Card 컴포넌트의 렌더링과 탭 전환 상호작용을 검증하는 통합 테스트 파일이다. Overview·Ingredients·Instructions 세 탭의 초기 렌더링, 탭 클릭 후 콘텐츠 전환, 아이콘 존재 여부를 확인한다. `whenSettled`를 사용해 비동기 상태 업데이트가 완료된 후 DOM 상태를 단언한다.

## 의존성

### 외부 패키지
- (없음)

### 저장소 내부 모듈
- [`../utils/test-utils`](../utils/test-utils.ts.md) — `loadExample`, `getSurface`, `getDeepTextContent`, `querySelectorAllDeep`, `whenSettled` 임포트
- [`../../src/local-gallery`](../../src/local-gallery.ts.md) — `LocalGallery` 클래스 임포트

## Exports

명시적 export 없음. Jasmine 스위트를 전역에 등록하는 side-effect만 존재한다.

## 동작 흐름

1. `beforeEach`: `loadExample('24_recipe-card.json')` → `getSurface` → `getDeepTextContent` 순으로 초기 상태를 수집한다.
2. `afterEach`: `gallery.remove()`로 DOM 정리.
3. 탭 전환 테스트에서는 `querySelectorAllDeep(surface, '.a2ui-tab-button')`으로 모든 탭 버튼을 조회하고, `Array.find`로 특정 탭을 찾아 `.click()` 후 `whenSettled`를 기다린 뒤 `getDeepTextContent`를 재호출해 새 콘텐츠를 단언한다.

## 테스트 케이스

### `should render tab titles`
- **검증 동작**: 초기 텍스트에 `'Overview'`, `'Ingredients'`, `'Instructions'` 세 탭 제목이 모두 포함되는지 확인한다.
- **픽스처**: `24_recipe-card.json`

### `should render Overview content`
- **검증 동작**: 초기 Overview 탭에서 요리 이름 `'Mediterranean Quinoa Bowl'`, 평점 `'4.9'`, 조리 시간 `'15 min prep'`·`'20 min cook'`·`'Serves 4'`가 렌더링되는지 확인한다.
- **픽스처**: 동일.

### `should render icon`
- **검증 동작**: `a2ui-icon` 요소가 최소 1개 존재(truthy)하는지 확인한다.
- **픽스처**: 동일.

### `should switch to Ingredients tab`
- **검증 동작**: `.a2ui-tab-button` 중 `'Ingredients'` 텍스트를 가진 탭을 찾아 클릭 → `whenSettled` → 재수집된 텍스트에 `'1 cup quinoa'`와 `'1 cucumber, diced'`가 포함되는지 확인한다.
- **모킹/유틸**: `whenSettled(gallery)`로 비동기 렌더링 완료 대기.

### `should switch to Instructions tab`
- **검증 동작**: `'Instructions'` 탭을 찾아 클릭 → `whenSettled` → `'Rinse quinoa and bring to a boil in water.'`와 `'Mix with diced vegetables.'`가 포함되는지 확인한다.
- **모킹/유틸**: `whenSettled(gallery)`.
