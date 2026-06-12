# renderers/angular/a2ui_explorer/src/app/tests/v0_8/05_product-card.spec.ts

## 개요

v0.8 프로토콜의 "Product Card (basic)" 예제의 렌더링 및 버튼 인터랙션을 검증하는 Jasmine 통합 테스트 파일이다. 상품명, 별점, 리뷰 수, 가격 텍스트가 올바르게 렌더링되는지, 그리고 "Add to Cart" 버튼 클릭 시 `addToCart` 액션이 이벤트 로그에 기록되는지 확인한다.

## 의존성

### 외부 패키지
- `@angular/core/testing`: `ComponentFixture`

### 저장소 내부 모듈
- [`../../demo.component`](../../demo.component.ts.md) — `DemoComponent`
- [`../utils/test_utils`](../utils/test_utils.ts.md) — `Version`, `getCanvas`, `loadExample`, `wait`

## Exports

없음 (spec 파일).

## 테스트 케이스

**describe**: `'Example: Product Card (basic) (v0.8)'`

**공유 변수**: `textContent: string`, `fixture: ComponentFixture<DemoComponent>`

**`beforeEach` 설정**:
- `loadExample('Product Card (basic)', Version.V0_8)`로 `fixture`를 초기화한다.
- `getCanvas().textContent`를 `textContent`에 저장한다.

**테스트 1: `'should render expected text content'`**
- 검증: `textContent`에 다음 내용이 모두 포함되는지 확인한다.
  - `'Add to Cart'` — 버튼 레이블
  - `'Wireless Headphones Pro'` — 상품명
  - `'★★★★★'` — 별점 아이콘 문자열
  - `'(2,847 reviews)'` — 리뷰 수 표시
  - `'$199.99'` — 할인 판매 가격
  - `'$249.99'` — 원래 정가

**테스트 2: `'should dispatch addToCart action on button click'`**
- 동작: `fixture.nativeElement.querySelectorAll('a2ui-button button')`으로 버튼 목록을 가져온다. 개수가 0보다 많은지 확인한 뒤 인덱스 0 버튼을 선택해 `.click()`을 호출하고 `fixture.detectChanges()` 후 `await wait(10)`으로 대기한다.
- 검증: `component.eventsLog.length > 0`이고 `eventsLog[0].action.name === 'addToCart'`인지 확인한다.

**사용 픽스처/모킹**: `loadExample`이 TestBed를 구성하며 실제 v0.8 렌더러를 사용한다. 버튼 클릭 후 `wait(10)` ms 대기로 비동기 액션 처리를 보장한다.
