# renderers/angular/a2ui_explorer/src/app/tests/v0_9/31_incremental-dashboard.spec.ts

## 개요

`Incremental Dashboard` 예제의 브라우저 통합 테스트 파일이다. Angular `DemoComponent`를 실제로 로드한 뒤 증분(incremental) 업데이트가 모두 완료된 최종 상태를 검증한다. 증분 렌더링 완료 대기 시간으로 500ms를 사용하며, 로딩 중 텍스트가 최종 완료 텍스트로 교체되었는지 확인한다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture` 타입

### 저장소 내부 모듈
- [`../../demo.component`](../../demo.component.ts.md) — `DemoComponent`
- [`../utils`](../utils/index.ts.md) — `getCanvas`, `loadExample`, `wait`

## Exports

이 파일은 테스트 스위트 정의만 포함하며, 외부로 내보내는 항목이 없다.

## 테스트 케이스 명세

### 픽스처 및 셋업

`describe('Example: Incremental Dashboard')` 블록 내에서 `fixture: ComponentFixture<DemoComponent>`와 `textContent: string`을 공유 변수로 선언한다.

`beforeEach` — 각 테스트 전에 비동기로 실행된다.
1. `loadExample('Incremental Dashboard')`를 호출하여 해당 예제를 DemoComponent에 로드하고 fixture를 얻는다.
2. `wait(500)` — 증분 업데이트가 모두 처리될 때까지 500ms 대기한다 (단순 로드보다 긴 대기가 필요한 이유를 주석으로 명시).
3. `fixture.detectChanges()` — Angular 변경 감지를 수동으로 트리거한다.
4. `getCanvas().textContent`로 렌더링된 캔버스 영역의 텍스트 전체를 `textContent`에 저장한다.

### 테스트 케이스 1: `should render header`

- **검증 대상**: 캔버스에 `'System Dashboard'` 문자열이 존재하는지 확인한다.
- **사용 픽스처**: `beforeEach`에서 설정한 `textContent`.

### 테스트 케이스 2: `should render final state content`

- **검증 대상 (긍정)**: 다음 4개의 최종 상태 텍스트가 모두 캔버스에 존재해야 한다.
  - `'Analytics are ready.'`
  - `'System boot complete.'`
  - `'All services healthy.'`
  - `'Waiting for user input.'`
- **검증 대상 (부정)**: 증분 로딩 중 텍스트가 캔버스에서 제거되었는지 확인한다.
  - `'Loading analytics...'` — 존재하지 않아야 한다.
  - `'Loading logs...'` — 존재하지 않아야 한다.
- **사용 픽스처**: `beforeEach`에서 500ms 대기 후 설정한 `textContent`.

## 동작 흐름

`loadExample` → `wait(500)` → `detectChanges` → `getCanvas().textContent` 순서로 셋업 후, 헤더 존재 여부와 증분 업데이트 최종 상태(로딩 텍스트 제거 + 완료 텍스트 표시)를 순차적으로 검증한다.
