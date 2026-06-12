# renderers/angular/a2ui_explorer/src/app/tests/v0_9/08_user-profile.spec.ts

## 개요

`User Profile` 예제 컴포넌트의 렌더링을 검증하는 Angular 통합 테스트 파일이다. 사용자 이름·핸들·소개·팔로워 관련 텍스트가 올바르게 표시되는지, 숫자 포맷이 맞는지, 그리고 프로필 이미지의 `src` 속성이 정확한 URL을 가리키는지를 확인한다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture` 타입

### 저장소 내부 모듈
- [`../../demo.component.ts`](../../demo.component.ts.md) — `DemoComponent`
- [`../utils`](../utils/index.ts.md) — `getCanvas`, `loadExample`

## Exports

이 파일은 Jasmine `describe` 블록을 최상위에서 실행하며 별도로 export하는 항목은 없다.

## 테스트 케이스

### `describe('Example: User Profile')`

픽스처 변수: `fixture: ComponentFixture<DemoComponent>`, `textContent: string`

**`beforeEach`**
- `loadExample('User Profile')`를 비동기로 호출하여 `fixture`를 얻는다.
- `getCanvas().textContent`로 캔버스 텍스트를 `textContent`에 저장한다.
- `wait`은 사용하지 않는다.

---

**`should render text content`**
- 검증 동작: 캔버스 텍스트에 `'Sarah Chen'`, `'@sarahchen'`, `'Product Designer at Tech Co. Creating delightful experiences.'`, `'Followers'`, `'Following'`, `'Posts'`, `'Follow'` 가 포함되어 있는지 확인한다.
- 픽스처/모킹: `beforeEach`에서 로드된 `textContent` 사용.

---

**`should render formatted numbers`**
- 검증 동작: 텍스트에 `'892'`, `'347'` 값이 포함되어 있는지 확인한다 (팔로워/팔로잉 숫자 렌더링 검증).
- 픽스처/모킹: `beforeEach`에서 로드된 `textContent` 사용.

---

**`should render image`**
- 검증 동작: `fixture.nativeElement.querySelector('img')`로 이미지 요소가 존재하는지 확인하고, `getAttribute('src')`가 정확히 `'https://images.unsplash.com/photo-1494790108377-be9c29b29330?w=150&h=150&fit=crop'`인지 검증한다.
- 픽스처/모킹: `fixture.nativeElement` DOM 직접 쿼리.

## 동작 흐름

`beforeEach`에서 `'User Profile'` 예제를 로드하고 텍스트를 추출한다. 각 `it` 블록은 서로 독립적으로 텍스트 내용, 숫자 포맷, 이미지 URL을 검증한다.
