# renderers/lit/a2ui_explorer/tests/v0_9/31_incremental-dashboard.test.ts

## 개요

`31_incremental-dashboard.json` 예제를 로드하여 Incremental Dashboard 컴포넌트의 증분 업데이트 최종 상태를 검증하는 통합 테스트 파일이다. 증분 렌더링 처리가 모두 완료된 뒤 최종 상태 텍스트가 나타나고, 중간 로딩 텍스트는 사라졌는지 확인한다. `beforeEach`에서 `wait(500)` 대기를 통해 모든 증분 업데이트가 처리될 충분한 시간을 확보한다.

## 의존성

### 외부 패키지
- (없음)

### 저장소 내부 모듈
- [`../utils/test-utils`](../utils/test-utils.ts.md) — `loadExample`, `getSurface`, `getDeepTextContent`, `whenSettled` 임포트
- [`../../src/local-gallery`](../../src/local-gallery.ts.md) — `LocalGallery` 클래스 임포트

## Exports

명시적 export 없음.

## 상세 명세

### `wait(ms: number): Promise<void>`
- `const`로 파일 상단에 정의된 인라인 헬퍼.
- `setTimeout` 기반 지연 프로미스. `beforeEach`에서 `await wait(500)`으로 호출되어 증분 업데이트 처리 완료를 기다린다.

## 동작 흐름

1. `beforeEach`: `loadExample('31_incremental-dashboard.json')` → `getSurface` → `wait(500)` (증분 업데이트 처리 대기) → `whenSettled` → `getDeepTextContent`.
2. `afterEach`: `gallery.remove()`.

## 테스트 케이스

### `should render header`
- **검증 동작**: 최종 텍스트에 `'System Dashboard'`가 포함되는지 확인한다.
- **픽스처**: `31_incremental-dashboard.json`

### `should render final state content`
- **검증 동작**:
  - 최종 완료 메시지 `'Analytics are ready.'`, `'System boot complete.'`, `'All services healthy.'`, `'Waiting for user input.'`가 모두 포함되는지 확인한다.
  - 중간 로딩 상태 텍스트 `'Loading analytics...'`와 `'Loading logs...'`는 포함되지 않는지(`not.toContain`) 확인한다.
- **픽스처**: 동일.
