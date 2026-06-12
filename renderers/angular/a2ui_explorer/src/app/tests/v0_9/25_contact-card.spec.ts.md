# renderers/angular/a2ui_explorer/src/app/tests/v0_9/25_contact-card.spec.ts

## 개요

`Contact Card` 예제 컴포넌트에 대한 Angular 통합 테스트 파일이다. 연락처 정보 텍스트, 프로필 이미지 URL, 아이콘 수와 종류, 그리고 `Call` / `Message` 버튼 클릭 인터랙션을 검증하는 5개의 테스트 케이스로 구성된다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture`

### 저장소 내부 모듈
- [`../../demo.component`](../../demo.component.ts.md) — `DemoComponent`
- [`../utils`](../utils/index.ts.md) — `getCanvas`, `loadExample`, `wait`

## Exports

직접 export하는 심볼 없음. Jasmine `describe` 블록으로 테스트 스위트를 등록한다.

## 테스트 케이스 명세

### `describe('Example: Contact Card', ...)`

#### 픽스처 / 모킹

- `fixture`: `ComponentFixture<DemoComponent>` — `loadExample('Contact Card')`로 생성.
- `component`: `DemoComponent` — `fixture.componentInstance`에서 획득.
- `textContent`: `string` — `getCanvas().textContent`.
- `beforeEach`는 `async`로 동작한다.

#### 테스트 케이스

1. **`should render text content`**
   - 검증 동작: 캔버스 텍스트에 `'David Park'`, `'Engineering Manager'`, `'+1 (555) 234-5678'`, `'david.park@company.com'`, `'San Francisco, CA'`, `'Call'`, `'Message'`가 포함되는지 확인한다.
   - 픽스처/모킹: `textContent`.

2. **`should render image`**
   - 검증 동작: `img` 태그가 DOM에 존재하는지 확인하고, `src` 속성이 `'https://images.unsplash.com/photo-1507003211169-0a1dd7228f2d?w=100&h=100&fit=crop'`인지 검증한다.
   - 픽스처/모킹: `fixture.nativeElement.querySelector('img')`.

3. **`should render icons`**
   - 검증 동작: `.a2ui-icon` 클래스 요소가 3개 이상인지 확인하고, 캔버스 텍스트에 `'location_on'`, `'phone'`, `'mail'` 아이콘 이름이 포함되는지 검증한다.
   - 픽스처/모킹: `fixture.nativeElement.querySelectorAll('.a2ui-icon')`, `textContent`.

4. **`should handle Call button click`**
   - 검증 동작: `.a2ui-button` 중 텍스트에 `'Call'`이 포함된 버튼을 찾아 클릭한다. `wait(50)` 후 변경 감지를 두 차례 수행한다. `component.eventsLog.length`가 1이고 `eventsLog[0].action.name`이 `'call'`인지 검증한다.
   - 픽스처/모킹: `fixture.nativeElement.querySelectorAll('.a2ui-button')`, `component.eventsLog`.

5. **`should handle Message button click`**
   - 검증 동작: `.a2ui-button` 중 텍스트에 `'Message'`가 포함된 버튼을 찾아 클릭한다. `wait(50)` 후 변경 감지를 두 차례 수행한다. `component.eventsLog.length`가 1이고 `eventsLog[0].action.name`이 `'message'`인지 검증한다.
   - 픽스처/모킹: `fixture.nativeElement.querySelectorAll('.a2ui-button')`, `component.eventsLog`.

## 동작 흐름

`beforeEach`에서 `'Contact Card'` 예제를 로드하고 텍스트 스냅샷을 설정한다. 정적 렌더링 검증(케이스 1~3)은 스냅샷 및 DOM 쿼리로 즉시 수행되고, 인터랙션 테스트(케이스 4~5)는 버튼 탐색 → 클릭 → 비동기 대기 → 변경 감지 → `eventsLog` 검증 순서를 따른다.
