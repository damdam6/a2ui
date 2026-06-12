# renderers/angular/a2ui_explorer/src/app/tests/v0_9/33_financial-data-grid.spec.ts

## 개요

`Financial Data Grid` 예제의 브라우저 통합 테스트 파일이다. 금융 데이터 그리드 렌더링 결과를 검증하며, 테이블 헤더·자산 이름·심볼·아이콘 표시 여부를 확인한다. 인터랙션 없이 초기 렌더링 상태만 검증한다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture` 타입

### 저장소 내부 모듈
- [`../../demo.component`](../../demo.component.ts.md) — `DemoComponent`
- [`../utils`](../utils/index.ts.md) — `getCanvas`, `loadExample`

## Exports

이 파일은 테스트 스위트 정의만 포함하며, 외부로 내보내는 항목이 없다.

## 테스트 케이스 명세

### 픽스처 및 셋업

`describe('Example: Financial Data Grid')` 블록에서 `fixture: ComponentFixture<DemoComponent>`와 `textContent: string`을 공유 변수로 선언한다.

`beforeEach` — 각 테스트 전에 비동기로 실행된다.
1. `loadExample('Financial Data Grid')`를 호출하여 예제를 로드하고 fixture를 얻는다.
2. `getCanvas().textContent`를 `textContent`에 저장한다 (`wait()` 미사용).

### 테스트 케이스 1: `should render table headers`

- **검증 대상**: 다음 테이블 헤더 텍스트들이 모두 캔버스에 존재해야 한다.
  - `'Asset'`
  - `'Price'`
  - `'24h Change'`
  - `'Market Cap'`

### 테스트 케이스 2: `should render asset names and symbols`

- **검증 대상**: 다음 자산 이름 및 심볼들이 모두 캔버스에 존재해야 한다.
  - `'Bitcoin'`, `'BTC'`
  - `'Ethereum'`, `'ETH'`
  - `'Solana'`, `'SOL'`

### 테스트 케이스 3: `should render icon`

- **검증 대상 (DOM)**: `fixture.nativeElement.querySelector('.a2ui-icon')`가 truthy이어야 한다 (DOM에 아이콘 요소 존재 확인).
- **검증 대상 (텍스트)**: `textContent`에 `'payment'`가 포함되어야 한다 (아이콘 이름 기반 텍스트 렌더링 확인).

## 동작 흐름

`loadExample` → `getCanvas().textContent` 셋업 후, 그리드 헤더 존재, 자산 데이터 렌더링, `.a2ui-icon` CSS 클래스를 가진 아이콘 요소 존재를 순차적으로 검증한다. 모든 테스트는 정적 렌더링 상태만 확인한다.
