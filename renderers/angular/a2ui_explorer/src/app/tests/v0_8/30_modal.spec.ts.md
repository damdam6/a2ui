# renderers/angular/a2ui_explorer/src/app/tests/v0_8/30_modal.spec.ts

## 개요

v0.8 버전의 "Modal Sample (basic)" 예제가 Angular 렌더러에서 올바르게 동작하는지 검증하는 Jasmine 테스트 스위트다. 초기 렌더링 텍스트 확인, 버튼 클릭으로 모달을 열었을 때의 오버레이 위치 스타일 검사, 그리고 버튼 클릭 시 `openModalEvent` 액션이 `eventsLog`에 기록되는지를 총 3개의 테스트로 검증한다. `ComponentFixture<DemoComponent>`를 직접 사용하여 인터랙션과 상태를 확인한다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture` 타입

### 저장소 내부 모듈
- [`../../demo.component`](../../demo.component.ts.md) — `DemoComponent` 클래스 (eventsLog 상태 보유)
- [`../utils/test_utils`](../utils/test_utils.ts.md) — `Version`, `getCanvas`, `loadExample`, `wait` 유틸리티 함수 제공

## Exports

이 파일은 Jasmine `describe` 블록을 최상위에 선언하며, TypeScript 모듈 export는 없다.

## 테스트 케이스 상세 명세

### `describe`: `'Example: Modal Sample (basic) (v0.8)'`

**픽스처 / 셋업**
- `textContent: string` — 각 테스트 전에 캔버스 DOM의 텍스트를 저장하는 변수.
- `fixture: ComponentFixture<DemoComponent>` — `loadExample` 반환값으로, DOM 쿼리 및 `detectChanges` 호출에 사용.
- `beforeEach`: `loadExample('Modal Sample (basic)', Version.V0_8)`를 `await`로 호출해 `fixture`를 얻고, `getCanvas().textContent`를 읽어 `textContent`에 저장한다.

---

#### 테스트 1: `'should render expected text content'`

초기 캔버스 텍스트에 `'Modal Component Sample'`과 `'Open Modal'`이 포함되는지 확인한다.

---

#### 테스트 2: `'should open modal and check overlay position'`

1. `fixture.nativeElement.querySelectorAll('a2ui-button button')`로 버튼 목록을 쿼리하고 1개 이상임을 검증한다.
2. 첫 번째 버튼(`buttons[0]`)을 `.click()`으로 클릭한 뒤 `fixture.detectChanges()`를 호출하고, `await wait(50)`으로 50ms 대기 후 다시 `detectChanges()`를 실행한다.
3. 갱신된 `getCanvas().textContent`에 `'This is the content inside the modal.'`이 포함됨을 확인한다.
4. `.a2ui-modal-overlay` 셀렉터로 오버레이 요소를 쿼리해 존재 여부를 검증한다.
5. `window.getComputedStyle(overlay).position`이 `'fixed'`임을 확인하여 오버레이가 뷰포트에 고정됨을 검증한다.

---

#### 테스트 3: `'should dispatch openModalEvent action on button click'`

1. `fixture.componentInstance`로 `component` 참조를 얻는다.
2. `querySelectorAll('a2ui-button button')`로 버튼을 조회하고 첫 번째 버튼(`btn`)이 존재함을 검증한다.
3. `btn.click()` 후 `fixture.detectChanges()`, `await wait(10)` 순서로 실행한다.
4. `component.eventsLog.length`가 0보다 큰지, 그리고 `component.eventsLog[0].action.name`이 `'openModalEvent'`인지 확인한다.

## 동작 흐름

`beforeEach` → 예제 로드 및 초기 텍스트 스냅샷 → 테스트 1(정적 텍스트) → 테스트 2(버튼 클릭 → 50ms 대기 → 모달 텍스트 및 `.a2ui-modal-overlay`의 `position: fixed` 검증) → 테스트 3(버튼 클릭 → 10ms 대기 → `eventsLog` 내 `openModalEvent` 기록 검증). 테스트 2와 3 모두 비동기 대기(`wait`) 후 `detectChanges`를 호출하는 패턴을 따른다.
