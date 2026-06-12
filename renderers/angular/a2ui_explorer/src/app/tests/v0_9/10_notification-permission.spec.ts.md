# renderers/angular/a2ui_explorer/src/app/tests/v0_9/10_notification-permission.spec.ts

## 개요

`Notification Permission` 예제 컴포넌트의 렌더링 및 버튼 인터랙션을 검증하는 Angular 통합 테스트 파일이다. 텍스트·아이콘 렌더링 외에도 Yes/No 버튼을 클릭했을 때 `DemoComponent`의 `eventsLog`에 각각 `'accept'`와 `'decline'` 액션이 기록되는지를 검증한다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture` 타입

### 저장소 내부 모듈
- [`../../demo.component.ts`](../../demo.component.ts.md) — `DemoComponent`
- [`../utils`](../utils/index.ts.md) — `getCanvas`, `loadExample`, `wait`

## Exports

이 파일은 Jasmine `describe` 블록을 최상위에서 실행하며 별도로 export하는 항목은 없다.

## 테스트 케이스

### `describe('Example: Notification Permission')`

픽스처 변수: `fixture: ComponentFixture<DemoComponent>`, `component: DemoComponent`, `textContent: string`

**`beforeEach`**
- `loadExample('Notification Permission')`를 비동기로 호출하여 `fixture`를 얻는다.
- `fixture.componentInstance`를 `component`에 저장한다.
- `getCanvas().textContent`로 캔버스 텍스트를 `textContent`에 저장한다.

---

**`should render text content`**
- 검증 동작: 텍스트에 `'Enable notification'`, `'Get alerts for order status changes'`, `'Yes'`, `'No'` 가 포함되어 있는지 확인한다.
- 픽스처/모킹: `beforeEach`에서 로드된 `textContent` 사용.

---

**`should render icon`**
- 검증 동작: `.a2ui-icon` 요소가 DOM에 존재하는지, 그리고 텍스트에 `'check'` 가 포함되어 있는지 확인한다.
- 픽스처/모킹: `fixture.nativeElement` DOM 쿼리 및 `textContent`.

---

**`should handle Yes button click`**
- 검증 동작: `.a2ui-button` 요소 배열에서 `textContent`에 `'Yes'`를 포함하는 버튼을 찾아 클릭한다. `fixture.detectChanges()` + `wait(50)` + `fixture.detectChanges()` 시퀀스 후 `component.eventsLog.length`가 `1`이고, `eventsLog[0].action.name`이 `'accept'`인지 검증한다.
- 픽스처/모킹: `fixture.nativeElement` DOM 쿼리, `wait(50)` 비동기 대기, `component.eventsLog` 상태 확인.

---

**`should handle No button click`**
- 검증 동작: `.a2ui-button` 요소 배열에서 `'No'`를 포함하는 버튼을 찾아 클릭한다. 동일한 변경 감지 시퀀스 후 `component.eventsLog[0].action.name`이 `'decline'`인지 검증한다.
- 픽스처/모킹: `fixture.nativeElement` DOM 쿼리, `wait(50)` 비동기 대기, `component.eventsLog` 상태 확인.

## 동작 흐름

`beforeEach`에서 예제를 로드하고 컴포넌트 인스턴스를 저장한다. 정적 렌더링 테스트 두 개는 텍스트/아이콘 존재를 확인하고, 인터랙션 테스트 두 개는 버튼 클릭 → `detectChanges` → `wait(50)` → `detectChanges` 패턴으로 액션 이벤트 로그를 검증한다.
