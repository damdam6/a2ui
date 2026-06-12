# renderers/angular/a2ui_explorer/src/app/tests/v0_8/20_restaurant-card.spec.ts

## 개요

v0.8 카탈로그의 "Restaurant Card (basic)" 예제가 Angular 렌더러에서 올바르게 렌더링되는지 검증하는 Jasmine 테스트 파일이다. 텍스트 내용, 이미지 src 속성, 아이콘 렌더링의 세 가지 측면을 각각 독립된 테스트 케이스로 검증한다. `ComponentFixture`를 활용하여 DOM 요소에 직접 접근한다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture`

### 저장소 내부 모듈
- [`../../demo.component`](../../demo.component.ts.md) — `DemoComponent`
- [`../utils/test_utils`](../utils/test_utils.ts.md) — `Version`, `getCanvas`, `loadExample`

## Exports

없음 (테스트 파일이므로 export 없음)

## 테스트 케이스 명세

### `describe`: `'Example: Restaurant Card (basic) (v0.8)'`

#### 픽스처 / 설정
- `fixture: ComponentFixture<DemoComponent>` — Angular 테스트 픽스처
- `textContent: string` — 캔버스 요소의 텍스트 내용
- `beforeEach`: `loadExample('Restaurant Card (basic)', Version.V0_8)`를 `await`하여 `fixture`에 저장하고, `getCanvas().textContent`를 `textContent`에 저장한다.

#### 테스트 케이스: `'should render expected text content'`
- **검증 동작**: 캔버스 텍스트에 다음 항목이 모두 포함되어 있는지 `toContain`으로 단언한다.
  - 식당명: `'The Italian Kitchen'`
  - 가격대: `'$$$'`
  - 카테고리: `'Italian • Pasta • Wine Bar'`
  - 평점/리뷰: `'4.8'`, `'(2,847 reviews)'`
  - 거리: `'0.8 mi'`
  - 배달 시간: `'25-35 min'`
- **사용 픽스처/모킹**: `textContent`

#### 테스트 케이스: `'should render image'`
- **검증 동작**: `fixture.nativeElement.querySelector('img')`로 img 요소를 찾아 truthy인지 확인하고, `getAttribute('src')`가 정확히 `'https://images.unsplash.com/photo-1517248135467-4c7edcad34c4?w=300&h=150&fit=crop'`인지 단언한다.
- **사용 픽스처/모킹**: `fixture`

#### 테스트 케이스: `'should render icon'`
- **검증 동작**: `fixture.nativeElement.querySelector('a2ui-icon')`로 아이콘 요소를 찾아 truthy인지 확인하고, `textContent`가 `'star'`인지 단언한다.
- **사용 픽스처/모킹**: `fixture`

## 동작 흐름

1. `beforeEach`에서 Restaurant Card 예제를 로드하여 `fixture`와 `textContent`를 초기화한다.
2. 첫 번째 테스트는 식당 정보 텍스트 전체를 순서대로 검증한다.
3. 두 번째 테스트는 DOM에서 `img` 태그를 찾아 이미지 URL이 Unsplash의 특정 사진 URL과 일치하는지 검증한다.
4. 세 번째 테스트는 `a2ui-icon` 커스텀 요소가 존재하고 그 텍스트 내용이 `'star'`임을 검증한다.
