# renderers/lit/a2ui_explorer/tests/v0_9/23_step-counter.test.ts

## 개요

`23_step-counter.json` 예제를 로드하여 Step Counter 컴포넌트의 렌더링을 검증하는 통합 테스트 파일이다. 텍스트 콘텐츠(걸음 수 관련 통계 레이블)와 아이콘 요소가 올바르게 렌더링되는지 확인한다. Jasmine `describe`/`it` 블록 구조를 사용하며, 각 테스트 전후 `LocalGallery` 인스턴스를 생성·제거하여 테스트 격리를 보장한다.

## 의존성

### 외부 패키지
- (없음; 테스트 프레임워크는 빌드 설정에서 전역 제공)

### 저장소 내부 모듈
- [`../utils/test-utils`](../utils/test-utils.ts.md) — `loadExample`, `getSurface`, `getDeepTextContent`, `querySelectorAllDeep` 임포트
- [`../../src/local-gallery`](../../src/local-gallery.ts.md) — `LocalGallery` 클래스 임포트

## Exports

이 파일은 명시적으로 export하는 항목이 없다. 테스트 파일이므로 side-effect(Jasmine 스위트 등록)만 존재한다.

## 동작 흐름

1. `beforeEach`: `loadExample('23_step-counter.json')`으로 갤러리 인스턴스를 생성하고, `getSurface`로 렌더링된 `HTMLElement`를 추출한 뒤 `getDeepTextContent`로 전체 텍스트를 수집한다.
2. `afterEach`: `gallery.remove()`로 DOM에서 컴포넌트를 제거해 다음 테스트와 격리한다.

## 테스트 케이스

### `should render text content`
- **검증 동작**: 수집한 `textContent`가 `"Today's Steps"`, `'Distance'`, `'Calories'` 세 문자열을 모두 포함하는지 확인한다.
- **픽스처/모킹**: `beforeEach`에서 로드된 `23_step-counter.json` 예제 파일.

### `should render icon`
- **검증 동작**: `querySelectorAllDeep`로 Shadow DOM 내부까지 탐색하여 첫 번째 `a2ui-icon` 요소가 존재(truthy)하는지, 그리고 텍스트에 아이콘 이름 `'person'`이 포함되는지 확인한다.
- **픽스처/모킹**: 동일한 예제 픽스처.
