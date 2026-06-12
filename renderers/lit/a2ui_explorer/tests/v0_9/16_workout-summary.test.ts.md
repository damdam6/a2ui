# renderers/lit/a2ui_explorer/tests/v0_9/16_workout-summary.test.ts

## 개요

`16_workout-summary.json` 예제 파일을 로드하여 "Workout Summary" 운동 요약 화면의 정적 렌더링을 검증하는 브라우저 통합 테스트 파일이다. 운동 완료 제목, 운동 시간, 칼로리, 거리 데이터 및 아이콘의 렌더링 정확성을 확인한다. 버튼 상호작용은 없으며 표시 데이터 검증에만 집중한다.

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

`loadExample('16_workout-summary.json')`을 비동기로 호출하여 `gallery`를 초기화한 후 `surface`와 `textContent`를 준비한다.

### afterEach

`gallery?.remove()`로 DOM을 정리한다.

## 테스트 케이스

### `should render text content`

- **검증 동작**: `textContent`에 다음 문자열들이 포함되어 있는지 확인한다: 제목 `'Workout Complete'`, 항목 라벨 `'Duration'`, 운동 시간 `'32:15'`, 항목 라벨 `'Calories'`, 칼로리 값 `'385'`, 항목 라벨 `'Distance'`, 거리 값 `'5.2 km'`.
- **픽스처/모킹**: `beforeEach`에서 로드된 `16_workout-summary.json` 예제 데이터.

### `should render icon`

- **검증 동작**: `querySelectorAllDeep(surface, 'a2ui-icon')[0]`이 truthy인지 확인하고, `textContent`에 아이콘 이름 `'directions_run'`이 포함되어 있는지 검증한다.
- **픽스처/모킹**: `querySelectorAllDeep` 유틸리티.
