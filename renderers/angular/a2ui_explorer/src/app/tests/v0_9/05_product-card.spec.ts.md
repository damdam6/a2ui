# renderers/angular/a2ui_explorer/src/app/tests/v0_9/05_product-card.spec.ts

## 개요

v0.9 버전의 "Product Card" 예제가 Angular 렌더러에서 올바르게 렌더링되고, "Add to Cart" 버튼 클릭 시 `addToCart` 액션이 `eventsLog`에 기록되는지 검증하는 Jasmine 테스트 스위트다. 텍스트 콘텐츠, 가격 포맷, 이미지 URL, 버튼 클릭 이벤트를 총 4개의 테스트로 검증한다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture` 타입

### 저장소 내부 모듈
- [`../../demo.component`](../../demo.component.ts.md) — `DemoComponent` 클래스 (eventsLog 상태 보유)
- [`../utils`](../utils/index.ts.md) — `getCanvas`, `loadExample`, `wait` 유틸리티 함수 제공

## Exports

이 파일은 Jasmine `describe` 블록을 최상위에 선언하며, TypeScript 모듈 export는 없다.

## 테스트 케이스 상세 명세

### `describe`: `'Example: Product Card'`

**픽스처 / 셋업**
- `fixture: ComponentFixture<DemoComponent>` — `loadExample` 반환값.
- `component: DemoComponent` — `fixture.componentInstance`.
- `textContent: string` — 각 테스트 전에 캔버스 DOM의 텍스트를 저장하는 변수.
- `beforeEach`: `loadExample('Product Card')`를 `await`로 호출하고, `fixture.componentInstance`를 `component`에 할당하며, `getCanvas().textContent`를 저장한다.

---

#### 테스트 1: `'should render text content'`

캔버스 텍스트에 다음이 포함되는지 확인한다.
- 상품명: `'Wireless Headphones Pro'`
- 별점: `'★★★★★'`
- 리뷰 수: `'2,847'`, `'reviews'`
- 버튼: `'Add to Cart'`

---

#### 테스트 2: `'should render formatted currency'`

캔버스 텍스트에 `'199.99'`(할인가)와 `'249.99'`(정가)가 포함되는지 확인한다. 화폐 기호 없이 숫자 부분만 검증한다.

---

#### 테스트 3: `'should render image'`

1. `fixture.nativeElement.querySelector('img')`로 이미지 요소를 조회하고 `truthy`인지 확인한다.
2. `img.getAttribute('src')`가 `'https://images.unsplash.com/photo-1505740420928-5e560c06d30e?w=300&h=200&fit=crop'`과 정확히 일치하는지 검증한다.

---

#### 테스트 4: `'should handle button click'`

1. `fixture.nativeElement.querySelector('.a2ui-button')`로 버튼 요소를 조회하고 `truthy`인지 확인한다.
2. `button.click()`, `fixture.detectChanges()`, `await wait(50)`, `fixture.detectChanges()` 순서로 실행한다.
3. `component.eventsLog.length === 1`이고 `eventsLog[0].action.name === 'addToCart'`인지 검증한다.

## 동작 흐름

`beforeEach` → 예제 로드 및 컴포넌트/텍스트 스냅샷 → 테스트 1(텍스트 콘텐츠) → 테스트 2(가격 포맷) → 테스트 3(이미지 src 속성 정확도) → 테스트 4(버튼 클릭 → 50ms 대기 → `eventsLog`의 `addToCart` 액션 검증).
