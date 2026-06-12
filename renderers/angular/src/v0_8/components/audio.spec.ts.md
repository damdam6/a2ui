# renderers/angular/src/v0_8/components/audio.spec.ts

## 개요

`AudioPlayer` Angular 컴포넌트에 대한 단위 테스트 파일이다. Angular `TestBed`를 사용하여 컴포넌트를 격리된 환경에서 마운트하고, DOM 렌더링 결과 및 테마 스타일 적용 여부를 검증한다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture`, `TestBed`

### 저장소 내부 모듈
- [`./audio`](./audio.ts.md) — `AudioPlayer` (테스트 대상)
- `../types` (타입 전용) — `AudioPlayerNode`
- [`../rendering/theming`](../rendering/theming.ts.md) — `Theme`
- [`../data/processor`](../data/processor.ts.md) — `MessageProcessor`
- [`../rendering/catalog`](../rendering/catalog.ts.md) — `Catalog`

## 테스트 케이스

### 픽스처 및 모킹 설정 (`beforeEach`)

- `mockProcessor`: `jasmine.createSpyObj`로 생성한 `MessageProcessor` 스파이 객체. `dispatch`, `resolvePath`, `getData` 메서드를 가짜로 구현한다.
- `mockTheme`: 실제 `Theme` 인스턴스를 생성한 후 `additionalStyles`를 `{AudioPlayer: {backgroundColor: 'red'}}`로 설정한다.
- `mockAudioNode`: `id: 'audio-1'`, `type: 'AudioPlayer'`, `weight: 1`, `properties.url.literalString: 'https://example.com/audio.mp3'`를 가진 `AudioPlayerNode` 픽스처.
- `TestBed.configureTestingModule`에 `AudioPlayer`를 `imports`로, `MessageProcessor`·`Theme`·`Catalog`를 프로바이더로 등록한다.
- `fixture.componentRef.setInput`으로 `surfaceId: 'surface-1'`, `component: mockAudioNode`, `weight: 1`, `url: mockAudioNode.properties.url`을 주입한다.
- `fixture.detectChanges()`로 초기 렌더링을 수행한다.

### 테스트: `'should create'`
- 검증: `component`가 truthy임을 확인하여 컴포넌트가 오류 없이 생성되는지 검증한다.

### 테스트: `'should render audio element with correct src'`
- 검증: DOM에서 `audio` 요소가 존재하는지, `src` 속성이 `'https://example.com/audio.mp3'`를 포함하는지, `controls` 속성이 `true`인지 확인한다.
- 사용 픽스처: `mockAudioNode.properties.url`의 `literalString` 값을 입력으로 사용.

### 테스트: `'should apply additional styles'`
- 검증: `audio` 요소의 `style.backgroundColor`가 `'red'`임을 확인하여 `theme.additionalStyles.AudioPlayer`가 올바르게 바인딩되는지 검증한다.
- 사용 모킹: `mockTheme.additionalStyles = {AudioPlayer: {backgroundColor: 'red'}}`.

## 동작 흐름

각 테스트는 `beforeEach`에서 독립적인 `TestBed` 환경을 구성하고, 픽스처를 통해 컴포넌트 입력을 설정한 후 `detectChanges()`로 렌더링을 트리거한다. 이후 `fixture.nativeElement.querySelector`로 DOM을 조회하여 기대값을 단언한다.
