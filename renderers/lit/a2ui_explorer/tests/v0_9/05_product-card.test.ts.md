# renderers/lit/a2ui_explorer/tests/v0_9/05_product-card.test.ts

## 개요

A2UI 예제 `05_product-card.json`을 `<local-gallery>`에 로드하여 상품 카드 UI가 올바르게 렌더링되고, "Add to Cart" 버튼 클릭 시 `gallery.actionLog`에 올바른 액션이 기록되는지 검증하는 통합 테스트 파일이다. 텍스트 콘텐츠, 가격 포맷, 이미지 `src` 속성, 버튼 인터랙션을 모두 검증한다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈
- [`../utils/test-utils`](../utils/test-utils.ts.md) — `loadExample`, `getSurface`, `getDeepTextContent`, `querySelectorAllDeep`, `whenSettled`
- [`../../src/local-gallery`](../../src/local-gallery.ts.md) — `LocalGallery` 타입

## Exports

없음 (테스트 파일)

## 테스트 케이스 명세

### `describe('Example: Product Card')`

**픽스처 설정:**
- `gallery: LocalGallery`, `surface: HTMLElement`, `textContent: string`을 스위트 범위 변수로 선언.
- `beforeEach`: `loadExample('05_product-card.json')`으로 갤러리를 마운트하고, `getSurface(gallery)`로 `a2ui-surface` 엘리먼트를 획득한 뒤, `getDeepTextContent(surface)`를 `textContent`에 저장.
- `afterEach`: `gallery?.remove()`로 DOM에서 갤러리를 제거.

---

**`it('should render text content')`**
- 검증 동작: 상품명, 별점 문자열, 리뷰 수, 리뷰 레이블, 버튼 텍스트가 렌더링된다.
- 기대 결과: `textContent`에 `'Wireless Headphones Pro'`, `'★★★★★'`, `'2,847'`, `'reviews'`, `'Add to Cart'` 포함.

**`it('should render formatted currency')`**
- 검증 동작: 현재 가격과 원래 가격이 올바르게 포맷되어 렌더링된다.
- 기대 결과: `textContent`에 `'199.99'`, `'249.99'` 포함.

**`it('should render image')`**
- 검증 동작: `<img>` 엘리먼트가 존재하고 올바른 `src` 속성 값을 갖는다.
- 구현: `querySelectorAllDeep(surface, 'img')[0]`으로 이미지 엘리먼트를 찾음.
- 기대 결과: `img`가 존재(truthy)하고 `img.getAttribute('src')`가 `'https://images.unsplash.com/photo-1505740420928-5e560c06d30e?w=300&h=200&fit=crop'`와 정확히 일치.

**`it('should handle button click')`**
- 검증 동작: `.a2ui-button` 클래스를 가진 첫 번째 버튼을 클릭하면 `gallery.actionLog`에 `name: 'addToCart'`인 액션이 1건 추가된다.
- 구현: `querySelectorAllDeep(surface, '.a2ui-button')[0]`으로 버튼을 찾아 `button.click()`을 호출한 후 `whenSettled(gallery)`를 await.
- 기대 결과: `gallery.actionLog.length === 1`, `gallery.actionLog[0].name === 'addToCart'`.

## 동작 흐름

정적 렌더링 검증 테스트들은 `beforeEach`에서 준비된 `textContent`와 `surface`를 사용한다. 이미지 검증은 `querySelectorAllDeep`으로 DOM 엘리먼트를 직접 찾아 속성을 확인한다. 버튼 클릭 테스트는 `querySelectorAllDeep`으로 버튼을 찾고, `click()` 후 `whenSettled`로 업데이트 완료를 기다린 뒤 `actionLog`를 검증한다.
