# renderers/angular/a2ui_explorer/src/app/tests/v0_9/20_restaurant-card.spec.ts

## 개요

`Restaurant Card` 예제 컴포넌트에 대한 Angular 통합 테스트 파일이다. 텍스트 렌더링, 이미지 src 속성, 아이콘 존재 여부를 검증하는 3개의 테스트 케이스로 구성된다. 버튼 인터랙션은 검증 범위에 포함되지 않으며, 시각적 표시 요소에 집중한다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture`

### 저장소 내부 모듈
- [`../../demo.component`](../../demo.component.ts.md) — `DemoComponent`
- [`../utils`](../utils/index.ts.md) — `getCanvas`, `loadExample`

## Exports

직접 export하는 심볼 없음. Jasmine `describe` 블록으로 테스트 스위트를 등록한다.

## 테스트 케이스 명세

### `describe('Example: Restaurant Card', ...)`

#### 픽스처 / 모킹

- `fixture`: `ComponentFixture<DemoComponent>` — `loadExample('Restaurant Card')`로 생성.
- `textContent`: `string` — `getCanvas().textContent`.
- `beforeEach`는 `async`로, 예제 로드 후 `textContent`를 갱신한다.
- `component` 인스턴스는 이 테스트에서 직접 사용하지 않는다(`wait`도 import하지 않음).

#### 테스트 케이스

1. **`should render text content`**
   - 검증 동작: 캔버스 텍스트에 `'The Italian Kitchen'`, `'$$$'`, `'Italian'`, `'4.8'`, `'2,847 reviews'`, `'0.8 mi'`, `'25-35 min'`이 모두 포함되는지 확인한다.
   - 픽스처/모킹: `textContent`.

2. **`should render image`**
   - 검증 동작: `fixture.nativeElement`에서 `img` 태그를 쿼리하여 존재 여부를 확인하고, `src` 속성이 정확히 `'https://images.unsplash.com/photo-1517248135467-4c7edcad34c4?w=300&h=150&fit=crop'`인지 검증한다.
   - 픽스처/모킹: `fixture.nativeElement.querySelector('img')`.

3. **`should render icon`**
   - 검증 동작: `.a2ui-icon` 클래스를 가진 요소가 DOM에 존재하는지 확인한다.
   - 픽스처/모킹: `fixture.nativeElement.querySelector('.a2ui-icon')`.

## 동작 흐름

`beforeEach`에서 `'Restaurant Card'` 예제를 로드하고 텍스트 스냅샷을 설정한다. 각 테스트는 렌더링 결과물의 특정 속성(텍스트, 이미지 URL, 아이콘 클래스)을 독립적으로 검증한다.
