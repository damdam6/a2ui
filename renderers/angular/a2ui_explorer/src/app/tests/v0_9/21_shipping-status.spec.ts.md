# renderers/angular/a2ui_explorer/src/app/tests/v0_9/21_shipping-status.spec.ts

## 개요

`Shipping Status` 예제 컴포넌트에 대한 Angular 통합 테스트 파일이다. `ComponentFixture`를 직접 사용하지 않고 `loadExample`의 반환값을 무시하여 텍스트 검증에만 집중한다. 배송 추적 UI의 텍스트 내용과 아이콘 이름을 검증하는 2개의 테스트 케이스로 구성된다.

## 의존성

### 외부 패키지
없음 (Angular Testing 모듈을 직접 import하지 않음).

### 저장소 내부 모듈
- [`../utils`](../utils/index.ts.md) — `getCanvas`, `loadExample`

## Exports

직접 export하는 심볼 없음. Jasmine `describe` 블록으로 테스트 스위트를 등록한다.

## 테스트 케이스 명세

### `describe('Example: Shipping Status', ...)`

#### 픽스처 / 모킹

- `textContent`: `string` — `getCanvas().textContent`.
- `beforeEach`는 `async`로, `loadExample('Shipping Status')`의 반환 픽스처를 변수에 저장하지 않고 대기만 한다. 완료 후 `textContent`를 캔버스에서 읽는다.

#### 테스트 케이스

1. **`should render text content`**
   - 검증 동작: 캔버스 텍스트에 `'Package Status'`, `'Tracking: 1Z999AA10123456784'`, `'Order Placed'`, `'Shipped'`, `'Out for Delivery'`, `'Delivered'`, `'Estimated delivery: Today by 8 PM'`가 모두 포함되는지 확인한다.
   - 픽스처/모킹: `textContent`.

2. **`should render icons`**
   - 검증 동작: 캔버스 텍스트에 Material 아이콘 이름인 `'info'`, `'check'`, `'send'`, `'calendar_today'`가 포함되는지 확인한다. (이 프로젝트에서는 아이콘 이름이 텍스트 노드로 렌더링됨을 전제로 한다.)
   - 픽스처/모킹: `textContent`.

## 동작 흐름

`beforeEach`에서 `'Shipping Status'` 예제를 비동기로 로드하고 캔버스 텍스트를 스냅샷한다. 두 테스트는 스냅샷된 텍스트에 대한 `toContain` 단언만 수행한다. `fixture`나 `component`를 직접 참조하지 않아 가장 단순한 구조의 테스트 파일에 해당한다.
