# renderers/lit/a2ui_explorer/tests/v0_9/27_stats-card.test.ts

## 개요

`27_stats-card.json` 예제를 로드하여 Stats Card 컴포넌트의 텍스트 및 아이콘 렌더링을 검증하는 통합 테스트 파일이다. 단일 `it` 블록에서 통계 카드 제목과 추세 아이콘 이름이 모두 렌더링되는지 확인한다. 의존하는 유틸리티 함수가 가장 적은 간단한 구조의 테스트다.

## 의존성

### 외부 패키지
- (없음)

### 저장소 내부 모듈
- [`../utils/test-utils`](../utils/test-utils.ts.md) — `loadExample`, `getSurface`, `getDeepTextContent` 임포트
- [`../../src/local-gallery`](../../src/local-gallery.ts.md) — `LocalGallery` 클래스 임포트

## Exports

명시적 export 없음.

## 동작 흐름

1. `beforeEach`: `loadExample('27_stats-card.json')` → `getSurface` → `getDeepTextContent`.
2. `afterEach`: `gallery.remove()`.

## 테스트 케이스

### `should render text content and icons`
- **검증 동작**: 텍스트에 아이콘 이름 `'trending_up'`, 카드 제목 `'Monthly Revenue'`, 변화 방향 아이콘 `'arrow_upward'`가 모두 포함되는지 확인한다.
- **픽스처**: `27_stats-card.json`
