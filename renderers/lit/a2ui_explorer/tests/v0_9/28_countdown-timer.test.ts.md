# renderers/lit/a2ui_explorer/tests/v0_9/28_countdown-timer.test.ts

## 개요

`28_countdown-timer.json` 예제를 로드하여 Countdown Timer 컴포넌트의 렌더링을 검증하는 통합 테스트 파일이다. 카운트다운 제목, 숫자 값(Days/Hours/Minutes), 목표 날짜 문자열이 올바르게 렌더링되는지 두 개의 `it` 블록으로 확인한다.

## 의존성

### 외부 패키지
- (없음)

### 저장소 내부 모듈
- [`../utils/test-utils`](../utils/test-utils.ts.md) — `loadExample`, `getSurface`, `getDeepTextContent` 임포트
- [`../../src/local-gallery`](../../src/local-gallery.ts.md) — `LocalGallery` 클래스 임포트

## Exports

명시적 export 없음.

## 동작 흐름

1. `beforeEach`: `loadExample('28_countdown-timer.json')` → `getSurface` → `getDeepTextContent`.
2. `afterEach`: `gallery.remove()`.

## 테스트 케이스

### `should render text content`
- **검증 동작**: 텍스트에 `'Product Launch'`, 숫자 `'14'`·`'08'`·`'32'`, 단위 레이블 `'Days'`·`'Hours'`·`'Minutes'`가 모두 포함되는지 확인한다.
- **픽스처**: `28_countdown-timer.json`

### `should render date`
- **검증 동작**: 텍스트에 목표 날짜를 나타내는 `'January'`와 `'2025'`가 포함되는지 확인한다.
- **픽스처**: 동일.
