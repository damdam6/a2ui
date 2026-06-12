# renderers/angular/a2ui_explorer/src/app/tests/v0_8/21_shipping-status.spec.ts

## 개요

v0.8 카탈로그의 "Shipping Status (basic)" 예제가 Angular 렌더러에서 올바르게 렌더링되는지 검증하는 Jasmine 테스트 파일이다. 배송 추적 UI에 표시되어야 하는 단계 레이블, 추적 번호, 배달 예정 시각 등의 텍스트를 단일 테스트 케이스로 확인한다. 인터랙션 테스트 없이 텍스트 렌더링 검증만 수행한다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈
- [`../utils/test_utils`](../utils/test_utils.ts.md) — `Version`, `getCanvas`, `loadExample`

## Exports

없음 (테스트 파일이므로 export 없음)

## 테스트 케이스 명세

### `describe`: `'Example: Shipping Status (basic) (v0.8)'`

#### 픽스처 / 설정
- `textContent: string` — 각 테스트 실행 전 캔버스 요소의 텍스트 내용
- `beforeEach`: `loadExample('Shipping Status (basic)', Version.V0_8)`를 `await`하여 예제를 로드한 후, `getCanvas().textContent`를 `textContent`에 저장한다.

#### 테스트 케이스: `'should render expected text content'`
- **검증 동작**: 캔버스 텍스트에 다음 항목이 모두 포함되어 있는지 `toContain`으로 단언한다.
  - 섹션 제목: `'Package Status'`
  - 배송 단계: `'Order Placed'`, `'Shipped'`, `'Out for Delivery'`, `'Delivered'`
  - 추적 번호: `'Tracking: 1Z999AA10123456784'`
  - 아이콘 텍스트: `'send'`
  - 예상 배달 시각: `'Estimated delivery: Today by 8 PM'`
- **사용 픽스처/모킹**: `textContent`, `loadExample`, `getCanvas`

## 동작 흐름

1. `beforeEach`에서 `loadExample`을 호출해 Shipping Status 예제를 로드한다.
2. `getCanvas().textContent`로 렌더링된 캔버스 DOM 전체 텍스트를 추출한다.
3. 단일 `it` 블록에서 배송 단계 4개, 추적 번호, 아이콘 텍스트, 예상 배달 시각이 모두 포함되는지 검증한다.
