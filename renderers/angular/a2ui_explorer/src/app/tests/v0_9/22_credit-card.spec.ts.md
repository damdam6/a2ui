# renderers/angular/a2ui_explorer/src/app/tests/v0_9/22_credit-card.spec.ts

## 개요

`Credit Card` 예제 컴포넌트에 대한 Angular 통합 테스트 파일이다. 카드 정보 텍스트(카드번호 마스킹, 카드 소유자명, 만료일 등)와 아이콘 렌더링을 검증하는 2개의 테스트 케이스로 구성된다. 버튼 인터랙션은 포함되지 않는다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture`

### 저장소 내부 모듈
- [`../../demo.component`](../../demo.component.ts.md) — `DemoComponent`
- [`../utils`](../utils/index.ts.md) — `getCanvas`, `loadExample`

## Exports

직접 export하는 심볼 없음. Jasmine `describe` 블록으로 테스트 스위트를 등록한다.

## 테스트 케이스 명세

### `describe('Example: Credit Card', ...)`

#### 픽스처 / 모킹

- `fixture`: `ComponentFixture<DemoComponent>` — `loadExample('Credit Card')`로 생성.
- `textContent`: `string` — `getCanvas().textContent`.
- `beforeEach`는 `async`로 동작하며, `component` 인스턴스는 이 파일에서 사용하지 않는다.

#### 테스트 케이스

1. **`should render text content`**
   - 검증 동작: 캔버스 텍스트에 `'CARD HOLDER'`, `'EXPIRES'`, `'VISA'`, `'•••• •••• •••• 4242'`(마스킹된 카드 번호), `'SARAH JOHNSON'`, `'09/27'`이 모두 포함되는지 확인한다.
   - 픽스처/모킹: `textContent`.

2. **`should render icon`**
   - 검증 동작: `.a2ui-icon` 클래스 요소가 DOM에 존재하는지 확인하고, 캔버스 텍스트에 Material 아이콘 이름 `'payment'`가 포함되는지 검증한다.
   - 픽스처/모킹: `fixture.nativeElement.querySelector('.a2ui-icon')`, `textContent`.

## 동작 흐름

`beforeEach`에서 `'Credit Card'` 예제를 로드하고 텍스트 스냅샷을 설정한다. 두 테스트는 각각 텍스트 노드와 아이콘 DOM 요소를 독립적으로 검증한다.
