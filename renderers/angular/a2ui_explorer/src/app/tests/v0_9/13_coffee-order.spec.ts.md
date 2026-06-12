# renderers/angular/a2ui_explorer/src/app/tests/v0_9/13_coffee-order.spec.ts

## 개요

`Coffee Order` 예제 컴포넌트의 렌더링 및 버튼 인터랙션을 검증하는 Angular 통합 테스트 파일이다. 주문 항목 텍스트·아이콘 렌더링을 확인하고, `'Purchase'` 버튼과 `'Add to cart'` 버튼을 각각 클릭했을 때 `eventsLog`에 `'purchase'`와 `'add_to_cart'` 액션이 기록되는지 검증한다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture` 타입

### 저장소 내부 모듈
- [`../../demo.component.ts`](../../demo.component.ts.md) — `DemoComponent`
- [`../utils`](../utils/index.ts.md) — `getCanvas`, `loadExample`, `wait`

## Exports

이 파일은 Jasmine `describe` 블록을 최상위에서 실행하며 별도로 export하는 항목은 없다.

## 테스트 케이스

### `describe('Example: Coffee Order')`

픽스처 변수: `fixture: ComponentFixture<DemoComponent>`, `component: DemoComponent`, `textContent: string`

**`beforeEach`**
- `loadExample('Coffee Order')`를 비동기로 호출하여 `fixture`를 얻는다.
- `fixture.componentInstance`를 `component`에 저장한다.
- `getCanvas().textContent`를 `textContent`에 저장한다.

---

**`should render text content`**
- 검증 동작: 텍스트에 `'Subtotal'`, `'Tax'`, `'Total'`, `'Purchase'`, `'Add to cart'`, `'Sunrise Coffee'`, `'Oat Milk Latte'`, `'Grande, Extra Shot'`, `'Chocolate Croissant'`, `'Warmed'` 가 포함되어 있는지 확인한다.
- 픽스처/모킹: `textContent` 사용.

---

**`should render icon`**
- 검증 동작: `.a2ui-icon` 요소가 DOM에 존재하는지, 텍스트에 `'favorite'`가 포함되어 있는지 확인한다.
- 픽스처/모킹: `fixture.nativeElement` DOM 쿼리 및 `textContent`.

---

**`should handle Purchase button click`**
- 검증 동작: `.a2ui-button` 배열에서 `'Purchase'`를 포함하는 버튼을 찾아 클릭한다. `fixture.detectChanges()` + `wait(50)` + `fixture.detectChanges()` 후 `component.eventsLog.length`가 `1`이고, `eventsLog[0].action.name`이 `'purchase'`인지 검증한다.
- 픽스처/모킹: `fixture.nativeElement` DOM 쿼리, `wait(50)` 비동기 대기, `component.eventsLog` 상태 확인.

---

**`should handle Add to cart button click`**
- 검증 동작: `.a2ui-button` 배열에서 `'Add to cart'`를 포함하는 버튼을 찾아 클릭한다. 동일한 변경 감지 시퀀스 후 `eventsLog[0].action.name`이 `'add_to_cart'`인지 검증한다.
- 픽스처/모킹: `fixture.nativeElement` DOM 쿼리, `wait(50)` 비동기 대기, `component.eventsLog` 상태 확인.

## 동작 흐름

`beforeEach`에서 예제를 로드하고 컴포넌트 인스턴스를 저장한다. 렌더링 테스트 두 개는 주문 항목 텍스트와 아이콘을 정적으로 검증하고, 인터랙션 테스트 두 개는 각각 다른 버튼에 대한 클릭 이벤트와 그 결과 액션 이름을 독립적으로 검증한다.
