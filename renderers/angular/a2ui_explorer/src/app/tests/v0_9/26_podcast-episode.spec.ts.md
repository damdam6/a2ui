# renderers/angular/a2ui_explorer/src/app/tests/v0_9/26_podcast-episode.spec.ts

## 개요

`Podcast Episode` 예제 컴포넌트에 대한 Angular 통합 테스트 파일이다. 팟캐스트 에피소드 카드의 텍스트 내용, 썸네일 이미지 URL, 오디오 플레이어 요소 및 src를 검증하는 3개의 테스트 케이스로 구성된다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture`

### 저장소 내부 모듈
- [`../../demo.component`](../../demo.component.ts.md) — `DemoComponent`
- [`../utils`](../utils/index.ts.md) — `getCanvas`, `loadExample`

## Exports

직접 export하는 심볼 없음. Jasmine `describe` 블록으로 테스트 스위트를 등록한다.

## 테스트 케이스 명세

### `describe('Example: Podcast Episode', ...)`

#### 픽스처 / 모킹

- `fixture`: `ComponentFixture<DemoComponent>` — `loadExample('Podcast Episode')`로 생성.
- `textContent`: `string` — `getCanvas().textContent`.
- `beforeEach`는 `async`로 동작한다. `wait`는 import하지 않는다.

#### 테스트 케이스

1. **`should render text content`**
   - 검증 동작: 캔버스 텍스트에 `'Tech Talk Daily'`, `'The Future of AI in Product Design'`, `'45 min'`, `'2024'`, `'How AI is transforming the way we design and build products.'`가 포함되는지 확인한다.
   - 픽스처/모킹: `textContent`.

2. **`should render image`**
   - 검증 동작: `img` 태그가 DOM에 존재하는지 확인하고, `src` 속성이 `'https://images.unsplash.com/photo-1478737270239-2f02b77fc618?w=100&h=100&fit=crop'`인지 검증한다.
   - 픽스처/모킹: `fixture.nativeElement.querySelector('img')`.

3. **`should render audio player`**
   - 검증 동작: `audio` 태그가 DOM에 존재하는지 확인하고, `src` 속성이 `'https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3'`인지 검증한다.
   - 픽스처/모킹: `fixture.nativeElement.querySelector('audio')`.

## 동작 흐름

`beforeEach`에서 `'Podcast Episode'` 예제를 로드하고 텍스트 스냅샷을 설정한다. 세 테스트는 각각 텍스트, 이미지, 오디오 요소를 독립적으로 검증하며, 인터랙션 없이 렌더링 결과만 확인한다.
