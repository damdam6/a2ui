# renderers/angular/a2ui_explorer/src/app/tests/v0_8/3_interactive_button.spec.ts

## 개요

v0.8 버전의 "Interactive Button (minimal)" 예제가 Angular 렌더러에서 올바르게 렌더링되고 클릭 이벤트를 정상적으로 디스패치하는지 검증하는 Jasmine 테스트 스위트다. 정적 텍스트 렌더링 확인 1건과 버튼 클릭 후 `eventsLog`에 `button_clicked` 액션이 기록되는지 확인하는 인터랙션 테스트 1건, 총 2개의 테스트로 구성된다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture` 타입

### 저장소 내부 모듈
- [`../../demo.component`](../../demo.component.ts.md) — `DemoComponent` 클래스 (eventsLog 상태 보유)
- [`../utils/test_utils`](../utils/test_utils.ts.md) — `Version`, `getCanvas`, `loadExample`, `wait` 유틸리티 함수 제공

## Exports

이 파일은 Jasmine `describe` 블록을 최상위에 선언하며, TypeScript 모듈 export는 없다.

## 테스트 케이스 상세 명세

### `describe`: `'Example: Interactive Button (minimal) (v0.8)'`

**픽스처 / 셋업**
- `textContent: string` — 각 테스트 전에 캔버스 DOM의 텍스트를 저장하는 변수.
- `fixture: ComponentFixture<DemoComponent>` — `loadExample` 반환값.
- `beforeEach`: `loadExample('Interactive Button (minimal)', Version.V0_8)`를 `await`로 호출해 `fixture`를 얻고, `getCanvas().textContent`를 `textContent`에 저장한다.

---

#### 테스트 1: `'should render expected text content'`

캔버스 텍스트에 `'Click the button below'`와 `'Click Me'`가 포함되는지 확인한다.

---

#### 테스트 2: `'should dispatch button_clicked action on button click'`

1. `fixture.componentInstance`로 `component` 참조를 얻는다.
2. `querySelectorAll('a2ui-button button')`으로 버튼 목록을 조회하고 1개 이상임을 검증한다.
3. 첫 번째 버튼(`btn`)이 `truthy`인지 확인한 뒤 `.click()`으로 클릭한다.
4. `fixture.detectChanges()` 후 `await wait(10)`으로 10ms 대기한다.
5. `component.eventsLog.length`가 0보다 크고, `eventsLog[0].action.name`이 `'button_clicked'`인지 검증한다.

**모킹**: 없음. `DemoComponent`의 실제 `eventsLog` 배열을 직접 검사한다.

## 동작 흐름

`beforeEach` → 예제 로드 및 텍스트 스냅샷 → 테스트 1(정적 텍스트 검증) → 테스트 2(버튼 클릭 → 10ms 비동기 대기 → `eventsLog` 내 `button_clicked` 액션 기록 검증).
