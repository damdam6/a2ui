# renderers/angular/a2ui_explorer/src/app/tests/v0_8/08_user-profile.spec.ts

## 개요

v0.8 버전의 "User Profile (basic)" 예제 컴포넌트가 Angular 렌더러에서 올바르게 동작하는지 검증하는 Jasmine 테스트 스위트다. 프로필 정보의 텍스트 렌더링과 "Follow" 버튼 클릭 시 `follow` 액션이 디스패치되는지를 두 개의 독립 테스트 케이스로 검증한다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture` 타입

### 저장소 내부 모듈
- [`../../demo.component`](../../demo.component.ts.md) — `DemoComponent` 클래스
- [`../utils/test_utils`](../utils/test_utils.ts.md) — `Version`, `getCanvas`, `loadExample`, `wait` 유틸리티

## Exports

이 파일은 `describe` 블록으로 테스트를 등록할 뿐 명시적으로 export하는 항목이 없다.

## 테스트 케이스 명세

### 스위트: `Example: User Profile (basic) (v0.8)`

**픽스처 및 공유 변수**
- `textContent: string` — 캔버스 요소의 텍스트 내용
- `fixture: ComponentFixture<DemoComponent>` — Angular 컴포넌트 픽스처

**`beforeEach`**
`loadExample('User Profile (basic)', Version.V0_8)`를 비동기로 호출하여 픽스처를 초기화하고, `getCanvas().textContent`로 렌더링된 텍스트를 추출해 `textContent`에 저장한다.

---

#### 테스트 1: `should render expected text content`
- **검증 동작**: 캔버스에 렌더링된 텍스트가 유저 프로필 카드에 필요한 모든 정보를 포함하는지 확인한다.
- **검증 항목**: `'Followers'`, `'Following'`, `'Posts'`(통계 레이블), `'Sarah Chen'`(이름), `'@sarahchen'`(핸들), `'Product Designer at Tech Co. Creating delightful experiences.'`(바이오), `'12.4K'`(팔로워 수), `'892'`(팔로잉 수), `'347'`(게시물 수), `'Follow'`(버튼 텍스트) 포함 여부를 `toContain`으로 각각 단언한다.
- **픽스처/모킹**: `loadExample` 호출 결과 반환된 `textContent` 사용.

---

#### 테스트 2: `should dispatch follow action on button click`
- **검증 동작**: `a2ui-button button` 셀렉터로 조회한 버튼 목록의 첫 번째 버튼(인덱스 0)을 클릭했을 때 `follow` 액션이 디스패치되는지 검증한다.
- **검증 항목**: 버튼 목록 길이가 0보다 크고, 첫 번째 버튼이 존재하며, 클릭 후 10ms 대기 뒤 `component.eventsLog[0].action.name`이 `'follow'`인지 단언한다.
- **픽스처/모킹**: `fixture.componentInstance`로 컴포넌트 인스턴스 접근, `wait(10)` 비동기 대기.

## 동작 흐름

`beforeEach`에서 v0.8 User Profile 예제를 로드하고 캔버스 텍스트를 가져온다. 텍스트 렌더링 테스트는 사용자의 이름, 핸들, 바이오, 팔로워/팔로잉/게시물 수치 및 레이블, Follow 버튼 텍스트 열 개 항목을 검증한다. 버튼 액션 테스트는 첫 번째 `a2ui-button button`을 클릭하고, `fixture.detectChanges()` 및 `wait(10)` 후 `eventsLog`에서 `follow` 액션명을 확인한다.
