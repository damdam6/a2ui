# renderers/angular/a2ui_explorer/src/app/tests/v0_9/30_live-invitation-builder.spec.ts

## 개요

`Live Invitation Builder` 예제 컴포넌트에 대한 Angular 통합 테스트 파일이다. 초대장 빌더 UI의 정적 렌더링(텍스트, 날짜, 이미지, 입력 필드) 검증과, 입력 및 칩 선택에 따른 라이브 프리뷰 실시간 업데이트를 검증하는 7개의 테스트 케이스로 구성된다. `waitForCondition`을 사용하는 비동기 DOM 폴링 패턴이 특징이다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture`

### 저장소 내부 모듈
- [`../../demo.component`](../../demo.component.ts.md) — `DemoComponent`
- [`../utils`](../utils/index.ts.md) — `getCanvas`, `loadExample`, `wait`, `waitForCondition`

## Exports

직접 export하는 심볼 없음. Jasmine `describe` 블록으로 테스트 스위트를 등록한다.

## 테스트 케이스 명세

### `describe('Example: Live Invitation Builder', ...)`

#### 픽스처 / 모킹

- `fixture`: `ComponentFixture<DemoComponent>` — `loadExample('Live Invitation Builder')`로 생성.
- `textContent`: `string` — `getCanvas().textContent` (초기 스냅샷).
- `beforeEach`는 `async`로 동작한다.

#### 헬퍼 함수

**`getLivePreview(): HTMLElement`**

- `fixture.nativeElement`에서 모든 DOM 요소를 선택한다.
- 텍스트가 정확히 `'Celebrating'`인 요소를 찾는다.
- 해당 요소의 3단계 상위 부모(`parentElement!.parentElement!.parentElement!`)를 라이브 프리뷰 컨테이너로 반환한다.
- 반환 전 `expect(livePreview).withContext('Should Live Preview card content wrapper').toBeTruthy()`로 유효성을 검증한다.

#### 테스트 케이스

1. **`should render text content`**
   - 검증 동작: `wait(100)` 후 변경 감지를 수행하고 `textContent`를 갱신한다. 캔버스에 `'Invitation Builder'`, `'Customize your invitation'`, `'Live Preview'`가 포함되는지 확인한다. `getLivePreview()`로 프리뷰 컨테이너를 가져와 `'Celebrating'`과 `'Location:'`이 포함되는지 검증한다.
   - 픽스처/모킹: `getCanvas()`, `getLivePreview()`.

2. **`should render date`**
   - 검증 동작: 캔버스 텍스트에 `'July 15'`와 `'2025'`가 포함되는지 확인한다.
   - 픽스처/모킹: `textContent`.

3. **`should render image`**
   - 검증 동작: `img` 태그가 DOM에 존재하는지 확인하고, `src`가 `'https://images.unsplash.com/photo-1511795409834-ef04bbd61622?w=400&h=200&fit=crop'`인지 검증한다.
   - 픽스처/모킹: `fixture.nativeElement.querySelector('img')`.

4. **`should render inputs`**
   - 검증 동작: `input` 요소가 2개 이상 존재하는지 확인한다.
   - 픽스처/모킹: `fixture.nativeElement.querySelectorAll('input')`.

5. **`should update preview when changing Event Name input`**
   - 검증 동작: `type === 'text'` 또는 `type`이 없는 `input` 요소를 필터링하여 첫 번째를 Event Name 입력으로, 두 번째를 Guest of Honor 입력으로 사용한다. Event Name에 `'Awesome Party'`, Guest에 `'Alex Johnson'`을 입력한 뒤 `new Event('input')`을 각각 디스패치한다. `wait(10)` 후 변경 감지를 수행하고, `getLivePreview().textContent`에 `'Awesome Party'`가 포함되는지 검증한다.
   - 픽스처/모킹: `fixture.nativeElement.querySelectorAll('input')`, `getLivePreview()`.

6. **`should update preview when changing Guest of Honor input`**
   - 검증 동작: 위 케이스와 동일한 방식으로 Event Name에 `'Summer Gala'`, Guest에 `'John Doe'`를 입력한다. `getLivePreview().textContent`에 `'John Doe'`가 포함되는지 검증한다.
   - 픽스처/모킹: `fixture.nativeElement.querySelectorAll('input')`, `getLivePreview()`.

7. **`should update preview when changing location`**
   - 검증 동작: `wait(100)` 후 변경 감지를 수행한다. `.a2ui-chip` 요소가 3개 이상인지 확인한다. 텍스트가 `'Grand Ballroom'`인 칩을 찾아 클릭하고, `waitForCondition` 폴링(내부에서 `fixture.detectChanges()` 후 `getLivePreview().textContent.includes('Location: ballroom')` 반환)이 `true`가 될 때까지 기다린다. 성공하면 `ballroomUpdated`가 `true`임을 검증한다. 이어서 텍스트가 `'Sunset Terrace'`인 칩을 찾아 클릭하고, 동일 방식으로 `getLivePreview().textContent.includes('Location: terrace')`가 `true`가 될 때까지 대기한 뒤 `terraceUpdated`를 검증한다.
   - 픽스처/모킹: `fixture.nativeElement.querySelectorAll('.a2ui-chip')`, `getLivePreview()`, `waitForCondition`.

## 동작 흐름

`beforeEach`에서 `'Live Invitation Builder'` 예제를 로드하고 초기 텍스트 스냅샷을 설정한다. 정적 렌더링 검증(케이스 1~4)은 스냅샷 및 DOM 쿼리로 처리된다. 인터랙션 테스트(케이스 5~7)는 입력 이벤트 디스패치 또는 칩 클릭 후 라이브 프리뷰 DOM 변경을 검증하며, 비동기 렌더링 전파를 처리하기 위해 `waitForCondition` 폴링 패턴을 사용한다.
