# renderers/angular/a2ui_explorer/src/app/tests/v0_8/01_flight-status.spec.ts

## 개요

v0.8 프로토콜의 "Flight Status" 예제가 Angular 렌더러에서 올바르게 렌더링되는지 검증하는 Jasmine 통합 테스트 파일이다. 항공편 번호·출발지·목적지 텍스트, 운항 상태 레이블, Material Icon 아이콘의 DOM 존재 여부를 확인한다. 버튼 인터랙션은 포함하지 않는 순수 렌더링 검증 파일이다.

## 의존성

### 외부 패키지
- `@angular/core/testing`: `ComponentFixture`

### 저장소 내부 모듈
- [`../../demo.component`](../../demo.component.ts.md) — `DemoComponent`
- [`../utils/test_utils`](../utils/test_utils.ts.md) — `Version`, `getCanvas`, `loadExample`

## Exports

없음 (spec 파일).

## 테스트 케이스

**describe**: `'Example: Flight Status (v0.8)'`

**공유 변수**: `fixture: ComponentFixture<DemoComponent>`, `textContent: string`

**`beforeEach` 설정**:
- `loadExample('Flight Status', Version.V0_8)`를 awaiting해 `fixture`를 초기화한다. 정확한 이름 매칭을 시도하며, 실패 시 `loadExample` 내부에서 `'Flight Status (basic)'` 또는 `'Flight Status (minimal)'` 변형을 순서대로 탐색한다.
- `getCanvas().textContent`를 `textContent`에 저장한다.

**테스트 1: `'should render flight details'`**
- 검증: `textContent`에 `'OS 87'`(항공편 번호), `'Vienna'`(출발지), `'→'`(방향 화살표), `'New York'`(목적지)이 모두 포함되는지 확인한다.

**테스트 2: `'should render labels'`**
- 검증: `textContent`에 `'Departs'`, `'Arrives'`, `'Status'`, `'On Time'`이 모두 포함되는지 확인한다. 출발·도착 레이블과 운항 상태가 정상 렌더링됨을 보증한다.

**테스트 3: `'should render icon'`**
- 검증: `fixture.nativeElement.querySelector('.g-icon')`으로 아이콘 요소를 찾아 존재 여부(`toBeTruthy()`)를 확인하고, 해당 요소의 `textContent.trim()`이 `'send'`인지 확인한다. Material Icon 문자열이 올바르게 렌더링됨을 검증한다.

**사용 픽스처/모킹**: `loadExample`이 내부적으로 TestBed를 구성하며 모킹 없이 실제 v0.8 렌더러를 사용한다. `getCanvas()`로 `.canvas-frame` DOM 요소를 조회한다.
