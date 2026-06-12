# renderers/angular/a2ui_explorer/src/app/tests/v0_9/19_software-purchase.spec.ts

## 개요

`Software Purchase` 예제 컴포넌트에 대한 Angular 통합 테스트 파일이다. `loadExample`을 통해 해당 예제를 로드한 뒤, 렌더링 결과물(텍스트, 통화 포맷, 버튼 인터랙션)을 검증한다. 총 4개의 테스트 케이스로 구성되어 있으며, 각 케이스는 `beforeEach`에서 픽스처를 초기화한다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture`

### 저장소 내부 모듈
- [`../../demo.component`](../../demo.component.ts.md) — `DemoComponent`
- [`../utils`](../utils/index.ts.md) — `getCanvas`, `loadExample`, `wait`

## Exports

이 파일은 Jasmine `describe` 블록을 통해 테스트 스위트를 등록하므로 직접 export하는 심볼은 없다.

## 테스트 케이스 명세

### `describe('Example: Software Purchase', ...)`

#### 픽스처 / 모킹

- `fixture`: `ComponentFixture<DemoComponent>` — `loadExample('Software Purchase')` 호출로 생성.
- `component`: `DemoComponent` — `fixture.componentInstance`에서 획득.
- `textContent`: `string` — `getCanvas().textContent`로 캔버스 전체 텍스트를 저장.
- `beforeEach`는 `async`로, `loadExample` 완료 후 `textContent`를 갱신한다.

#### 테스트 케이스

1. **`should render text content`**
   - 검증 동작: 캔버스 텍스트에 `'Purchase License'`, `'Design Suite Pro'`, `'Number of seats'`, `'10 seats'`, `'Billing period'`, `'Annual'`, `'Monthly'`, `'Total'`, `'Confirm Purchase'`, `'Cancel'` 문자열이 포함되는지 확인한다.
   - 픽스처/모킹: `textContent` (beforeEach에서 설정).

2. **`should render formatted currency`**
   - 검증 동작: 캔버스 텍스트에 `'1,188'`(쉼표를 포함한 통화 포맷)이 포함되는지 확인한다.
   - 픽스처/모킹: `textContent`.

3. **`should handle Confirm Purchase button click`**
   - 검증 동작: `.a2ui-button` 클래스를 가진 버튼 중 텍스트가 `'Confirm Purchase'`를 포함하는 버튼을 찾아 클릭한다. `wait(50)` 대기 후 `fixture.detectChanges()`를 두 차례 호출한다. `component.eventsLog`의 길이가 1이고, `eventsLog[0].action.name`이 `'confirm'`인지 검증한다.
   - 픽스처/모킹: `fixture.nativeElement.querySelectorAll('.a2ui-button')`, `component.eventsLog`.

4. **`should handle Cancel button click`**
   - 검증 동작: `.a2ui-button` 중 텍스트가 `'Cancel'`을 포함하는 버튼을 찾아 클릭한다. `wait(50)` 후 변경 감지를 두 차례 수행한다. `component.eventsLog.length`가 1이고 `eventsLog[0].action.name`이 `'cancel'`인지 검증한다.
   - 픽스처/모킹: `fixture.nativeElement.querySelectorAll('.a2ui-button')`, `component.eventsLog`.

## 동작 흐름

`beforeEach`에서 `'Software Purchase'` 예제를 로드하고 캔버스 텍스트를 스냅샷한다. 각 테스트는 독립적으로 실행되며, 인터랙션 테스트(케이스 3, 4)는 DOM 쿼리 → 클릭 → 비동기 대기 → 변경 감지 → `eventsLog` 검증의 순서를 따른다.
