# renderers/angular/src/v0_8/components/video.spec.ts

## 개요

`Video` Angular 컴포넌트의 단위 테스트 파일이다. `url` 입력 값 유무에 따른 조건부 렌더링, `<video>` 요소의 `src`·`controls` 속성 설정, 테마 클래스 및 추가 스타일 적용을 검증한다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture`, `TestBed`

### 저장소 내부 모듈
- [`./video`](./video.ts.md) — 테스트 대상 컴포넌트 `Video`
- [`../types`](../types.ts.md) — 타입 `VideoNode`
- [`../rendering/theming`](../rendering/theming.ts.md) — `Theme` (mock)
- [`../data/processor`](../data/processor.ts.md) — `MessageProcessor` (spy mock)
- [`../rendering/catalog`](../rendering/catalog.ts.md) — `Catalog` (mock)

## Exports

없음 (테스트 파일).

## 테스트 케이스 명세

### 픽스처 및 모킹 설정 (`beforeEach`)

- `mockVideoNode`: `VideoNode` 타입. `{id: 'video-1', type: 'Video', weight: 1, properties: {url: {literalString: 'https://example.com/video.mp4'}}}`.
- `mockProcessor`: `jasmine.createSpyObj`로 `dispatch`, `resolvePath`, `getData` 스파이 등록.
- `mockTheme.components`: `{Video: {'vid-class': true}}` (any 캐스팅).
- `mockTheme.additionalStyles`: `{Video: {borderColor: 'blue'}}`.
- 컴포넌트 입력값: `surfaceId='surface-1'`, `component=mockVideoNode`, `weight=1`, `url=mockVideoNode.properties.url`.

---

### `should create`

컴포넌트 인스턴스가 생성되는지 확인한다.

---

### `should render video element with correct src when url is provided`

- `fixture.nativeElement.querySelector('video')`로 `<video>` 요소를 조회.
- `videoEl.src`에 `'https://example.com/video.mp4'`가 포함되는지 확인.
- `videoEl.controls`가 `true`인지 확인.

---

### `should apply theme classes and styles to section`

- `<section>` 요소의 `className`에 `'vid-class'`가 포함되는지 검증.
- `sectionEl.style.borderColor`가 `'blue'`인지 검증.

---

### `should not render anything if url is null`

- `url` 입력을 `null`로 변경 후 `detectChanges()`.
- `<section>` 요소가 `null`인지 확인하여 조건부 렌더링 블록(`@if`)이 올바르게 작동하는지 검증.

## 동작 흐름

`beforeEach`에서 `url`을 실제 URL 문자열로 설정하여 기본 렌더링을 확인하고, 개별 테스트에서 `null` 전환 시 DOM 요소가 완전히 제거되는 조건부 렌더링 경로를 검증한다.
