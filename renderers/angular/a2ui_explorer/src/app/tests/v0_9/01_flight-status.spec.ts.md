# renderers/angular/a2ui_explorer/src/app/tests/v0_9/01_flight-status.spec.ts

## 개요

v0.9 버전의 "Flight Status" 예제가 Angular 렌더러에서 올바르게 렌더링되는지 검증하는 Jasmine 테스트 스위트다. 항공편 정보(편명, 출발·도착 도시, 방향 기호), 레이블 및 상태 텍스트, 아이콘 요소의 세 측면을 3개의 테스트로 나눠 검증한다. v0.9 계열 테스트로, `loadExample` 호출 시 `Version` 인자를 생략하는 방식(기본값 사용)을 따른다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture` 타입

### 저장소 내부 모듈
- [`../../demo.component`](../../demo.component.ts.md) — `DemoComponent` 클래스
- [`../utils`](../utils/index.ts.md) — `getCanvas`, `loadExample` 유틸리티 함수 제공 (index.ts를 통한 배럴 import)

## Exports

이 파일은 Jasmine `describe` 블록을 최상위에 선언하며, TypeScript 모듈 export는 없다.

## 테스트 케이스 상세 명세

### `describe`: `'Example: Flight Status'`

**픽스처 / 셋업**
- `fixture: ComponentFixture<DemoComponent>` — `loadExample` 반환값.
- `textContent: string` — 각 테스트 전에 캔버스 DOM의 텍스트를 저장하는 변수.
- `beforeEach`: `loadExample('Flight Status')`를 `await`로 호출해 `fixture`를 얻고, `getCanvas().textContent`를 `textContent`에 저장한다.

---

#### 테스트 1: `'should render flight details'`

캔버스 텍스트에 다음이 포함되는지 확인한다.
- 편명: `'OS 87'`
- 출발지: `'Vienna'`
- 방향 기호: `'→'`
- 도착지: `'New York'`

---

#### 테스트 2: `'should render labels'`

캔버스 텍스트에 다음이 포함되는지 확인한다.
- `'Departs'`, `'Arrives'`, `'Status'`, `'On Time'`

---

#### 테스트 3: `'should render icon'`

1. `fixture.nativeElement.querySelector('.a2ui-icon')`로 아이콘 요소(`iconInnerEl`)를 조회하고 `truthy`인지 확인한다.
2. `iconInnerEl.textContent.trim()`이 `'send'`인지 검증하여 Material 아이콘 리가처가 올바르게 렌더링되었는지 확인한다.

## 동작 흐름

`beforeEach` → `loadExample`로 예제 초기화 → 텍스트 스냅샷 → 테스트 1(항공편 세부 정보 텍스트) → 테스트 2(레이블/상태 텍스트) → 테스트 3(`.a2ui-icon` DOM 요소 존재 및 텍스트 내용 검증).
