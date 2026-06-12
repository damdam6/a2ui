# renderers/angular/src/v0_9/catalog/basic/audio-player.component.spec.ts

## 개요

v0.9 `AudioPlayerComponent`의 단위 테스트 파일이다. `A2uiRendererService`와 `ComponentBinder`를 mock으로 대체하고, `setComponentProps` / `createBoundProperty` 유틸리티를 사용하여 컴포넌트 입력을 설정한 뒤, 오디오 요소와 설명 텍스트 렌더링을 검증한다.

## 의존성

### 외부 패키지

- `@angular/core/testing` — `ComponentFixture`, `TestBed`

### 저장소 내부 모듈

- [`./audio-player.component`](audio-player.component.ts.md) — `AudioPlayerComponent`
- `../../core/a2ui-renderer.service` — `A2uiRendererService`
- `../../core/component-binder.service` — `ComponentBinder`
- `../../core/test-utils` — `setComponentProps`, `createBoundProperty`, `ComponentToProps`

## 픽스처 및 모킹

### `mockRendererService`

```js
{
  surfaceGroup: {
    getSurface: jasmine.createSpy('getSurface').and.returnValue({
      componentsModel: new Map(),
      catalog: { id: 'mock-catalog', components: new Map() },
    }),
  },
}
```

`A2uiRendererService`를 대체하는 객체. `getSurface`는 빈 컴포넌트 모델과 mock 카탈로그를 가진 서피스 객체를 반환한다.

### `mockBinder`

`jasmine.createSpyObj('ComponentBinder', ['bind'])` — `ComponentBinder`의 `bind` 메서드를 spy로 대체.

### `defaultProps`

```js
{
  url: createBoundProperty('https://example.com/audio.mp3'),
  description: createBoundProperty('Test Audio'),
}
```

`ComponentToProps<AudioPlayerComponent>` 타입. 각 속성이 bound property 형태로 래핑되어 있다.

### 초기화 (`beforeEach` × 2)

첫 번째 `beforeEach`: `TestBed.configureTestingModule`에 `AudioPlayerComponent` 임포트, `A2uiRendererService`와 `ComponentBinder` mock 제공 후 `compileComponents`.

두 번째 `beforeEach`:
- `fixture = TestBed.createComponent(AudioPlayerComponent)` 생성.
- `fixture.componentRef.setInput('surfaceId', 'test-surface')`, `setInput('dataContextPath', '/')` 설정.
- `setComponentProps(fixture, defaultProps)` 호출로 `url`과 `description` 설정.

---

## 테스트 케이스

**`should create`**
- 검증 동작: `AudioPlayerComponent` 인스턴스가 정상 생성된다.
- `fixture.detectChanges()` 후 `expect(component).toBeTruthy()` 검증.

**`should render audio with url`**
- 검증 동작: `defaultProps`의 `url`이 설정된 경우 `<audio>` 요소가 렌더링되고, `.a2ui-audio-description` 요소의 텍스트가 `'Test Audio'`이다.
- `fixture.detectChanges()` 후 `querySelector('audio').src`가 truthy, `querySelector('.a2ui-audio-description').textContent.trim() === 'Test Audio'` 검증.

**`should not render description if not provided`**
- 검증 동작: `description` 속성을 `createBoundProperty(undefined)`로 설정하면 `.a2ui-audio-description` 요소가 DOM에 존재하지 않는다.
- `setComponentProps(fixture, { ...defaultProps, description: createBoundProperty(undefined) })` 후 `detectChanges`, `querySelector('.a2ui-audio-description')` 결과가 `null`이어야 한다.

**`should handle missing props`**
- 검증 동작: props를 빈 객체(`{}`)로 설정하면 `<audio>` 요소의 `src` 속성이 falsy(null)이다.
- `setComponentProps(fixture, {})` 후 `detectChanges`, `audio.getAttribute('src')` 결과가 falsy 검증.
