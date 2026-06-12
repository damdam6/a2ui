# renderers/angular/a2ui_explorer/src/app/tests/v0_9/11_purchase-complete.spec.ts

## 개요

`Purchase Complete` 예제 컴포넌트의 렌더링 및 버튼 인터랙션을 검증하는 Angular 통합 테스트 파일이다. 구매 완료 화면의 텍스트·통화 값·이미지·아이콘 렌더링을 확인하고, `'View Order Details'` 버튼 클릭 시 `eventsLog`에 `'view_details'` 액션이 기록되는지 검증한다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture` 타입

### 저장소 내부 모듈
- [`../../demo.component.ts`](../../demo.component.ts.md) — `DemoComponent`
- [`../utils`](../utils/index.ts.md) — `getCanvas`, `loadExample`, `wait`

## Exports

이 파일은 Jasmine `describe` 블록을 최상위에서 실행하며 별도로 export하는 항목은 없다.

## 테스트 케이스

### `describe('Example: Purchase Complete')`

픽스처 변수: `fixture: ComponentFixture<DemoComponent>`, `component: DemoComponent`, `textContent: string`

**`beforeEach`**
- `loadExample('Purchase Complete')`를 비동기로 호출하여 `fixture`를 얻는다.
- `fixture.componentInstance`를 `component`에 저장한다.
- `getCanvas().textContent`를 `textContent`에 저장한다.

---

**`should render text content`**
- 검증 동작: 텍스트에 `'Purchase Complete'`, `'Wireless Headphones Pro'`, `'Arrives Dec 18 - Dec 20'`, `'Sold by:'`, `'TechStore Official'`, `'View Order Details'`, `'199.99'` 가 포함되어 있는지 확인한다.
- 픽스처/모킹: `textContent` 사용.

---

**`should render image`**
- 검증 동작: `querySelector('img')`로 이미지 요소가 존재하는지 확인하고, `getAttribute('src')`가 정확히 `'https://images.unsplash.com/photo-1505740420928-5e560c06d30e?w=100&h=100&fit=crop'`인지 검증한다.
- 픽스처/모킹: `fixture.nativeElement` DOM 쿼리.

---

**`should render icons`**
- 검증 동작: 텍스트에 아이콘 리터럴 `'check'`와 `'arrow_forward'`가 포함되어 있는지 확인한다.
- 픽스처/모킹: `textContent` 사용.

---

**`should handle button click`**
- 검증 동작: `querySelector('.a2ui-button')`로 첫 번째 버튼을 찾아 클릭한다. `fixture.detectChanges()` + `wait(50)` + `fixture.detectChanges()` 후 `component.eventsLog.length`가 `1`이고, `eventsLog[0].action.name`이 `'view_details'`인지 검증한다.
- 픽스처/모킹: `fixture.nativeElement` DOM 쿼리, `wait(50)` 비동기 대기, `component.eventsLog` 상태 확인.

## 동작 흐름

`beforeEach`에서 예제를 로드하고 컴포넌트 인스턴스를 저장한다. 렌더링 테스트 세 개는 텍스트/이미지/아이콘을 정적으로 검증하고, 마지막 테스트는 버튼 클릭 인터랙션과 그 결과로 기록되는 액션 이름을 검증한다.
