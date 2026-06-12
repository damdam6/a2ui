# renderers/lit/a2ui_explorer/tests/v0_9/33_financial-data-grid.test.ts

## 개요

`33_financial-data-grid.json` 예제를 로드하여 Financial Data Grid 컴포넌트의 렌더링을 검증하는 통합 테스트 파일이다. 테이블 헤더(열 이름), 자산 이름 및 심볼, 아이콘 요소의 존재와 종류를 세 개의 `it` 블록으로 확인한다.

## 의존성

### 외부 패키지
- (없음)

### 저장소 내부 모듈
- [`../utils/test-utils`](../utils/test-utils.ts.md) — `loadExample`, `getSurface`, `getDeepTextContent`, `querySelectorAllDeep` 임포트
- [`../../src/local-gallery`](../../src/local-gallery.ts.md) — `LocalGallery` 클래스 임포트

## Exports

명시적 export 없음.

## 동작 흐름

1. `beforeEach`: `loadExample('33_financial-data-grid.json')` → `getSurface` → `getDeepTextContent`.
2. `afterEach`: `gallery.remove()`.

## 테스트 케이스

### `should render table headers`
- **검증 동작**: 텍스트에 열 헤더 `'Asset'`, `'Price'`, `'24h Change'`, `'Market Cap'`이 모두 포함되는지 확인한다.
- **픽스처**: `33_financial-data-grid.json`

### `should render asset names and symbols`
- **검증 동작**: 텍스트에 자산 전체 이름 `'Bitcoin'`·`'Ethereum'`·`'Solana'` 및 심볼 `'BTC'`·`'ETH'`·`'SOL'`이 모두 포함되는지 확인한다.
- **픽스처**: 동일.

### `should render icon`
- **검증 동작**: 첫 번째 `a2ui-icon` 요소가 존재(truthy)하고 텍스트에 아이콘 이름 `'payment'`가 포함되는지 확인한다.
- **픽스처**: 동일.
