# renderers/angular/a2ui_explorer/src/app/tests/v0_9/29_movie-card.spec.ts

## 개요

`Movie Card` 예제 컴포넌트에 대한 Angular 통합 테스트 파일이다. 영화 정보 텍스트, 포스터 이미지, 아이콘, 그리고 트레일러 모달 열기/닫기 인터랙션을 검증하는 4개의 테스트 케이스로 구성된다. 모달 안에서 `video` 요소의 src까지 검증하는 점이 특징이다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture`

### 저장소 내부 모듈
- [`../../demo.component`](../../demo.component.ts.md) — `DemoComponent`
- [`../utils`](../utils/index.ts.md) — `getCanvas`, `loadExample`, `wait`

## Exports

직접 export하는 심볼 없음. Jasmine `describe` 블록으로 테스트 스위트를 등록한다.

## 테스트 케이스 명세

### `describe('Example: Movie Card', ...)`

#### 픽스처 / 모킹

- `fixture`: `ComponentFixture<DemoComponent>` — `loadExample('Movie Card')`로 생성.
- `textContent`: `string` — `getCanvas().textContent`.
- `beforeEach`는 `async`로 동작한다.

#### 테스트 케이스

1. **`should render text content`**
   - 검증 동작: 캔버스 텍스트에 `'Interstellar'`, `'(2014)'`, `'Sci-Fi • Adventure • Drama'`, `'8.7/10'`, `'2h 49min'`, `'Watch Trailer'`가 포함되는지 확인한다.
   - 픽스처/모킹: `textContent`.

2. **`should render image`**
   - 검증 동작: `img` 태그가 DOM에 존재하는지 확인하고, `src` 속성이 `'https://images.unsplash.com/photo-1536440136628-849c177e76a1?w=200&h=300&fit=crop'`인지 검증한다.
   - 픽스처/모킹: `fixture.nativeElement.querySelector('img')`.

3. **`should render icons`**
   - 검증 동작: 캔버스 텍스트에 Material 아이콘 이름 `'calendar_today'`와 `'star'`가 포함되는지 확인한다.
   - 픽스처/모킹: `textContent`.

4. **`should open modal with video when clicking watch trailer button`**
   - 검증 동작: `.a2ui-modal-trigger` 클래스 요소를 찾아 클릭한다. `wait(100)` 후 변경 감지를 수행한다. `.a2ui-modal-overlay` 클래스 요소가 DOM에 나타났는지 확인한다. `video` 태그가 존재하고 `src`가 `'https://www.w3schools.com/html/mov_bbb.mp4'`인지 검증한다. 마지막으로 클린업 단계에서 `.a2ui-modal-close` 버튼을 클릭하고 `wait(100)` 후 변경 감지를 수행하여 모달을 닫는다.
   - 픽스처/모킹: `fixture.nativeElement.querySelector('.a2ui-modal-trigger')`, `.a2ui-modal-overlay`, `video`, `.a2ui-modal-close`.

## 동작 흐름

`beforeEach`에서 `'Movie Card'` 예제를 로드하고 텍스트 스냅샷을 설정한다. 정적 렌더링 검증(케이스 1~3)은 스냅샷 및 DOM 쿼리로 즉시 수행된다. 모달 인터랙션 테스트(케이스 4)는 트리거 버튼 클릭 → 비동기 대기 → 모달 오버레이 확인 → 비디오 src 검증 → 모달 닫기 정리 순서를 따른다.
