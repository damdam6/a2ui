# renderers/angular/src/v0_9/catalog/basic/image.component.spec.ts

## 개요

`ImageComponent`에 대한 Angular 단위 테스트 파일이다. 컴포넌트 생성, URL 기반 이미지 렌더링, `alt` 텍스트 바인딩, `object-fit` 스타일 적용, 모든 지원 variant CSS 클래스 적용 동작을 검증한다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture`, `TestBed`

### 저장소 내부 모듈
- [`./image.component`](./image.component.ts.md) — `ImageComponent`
- [`../../core/a2ui-renderer.service`](../../core/a2ui-renderer.service.ts.md) — `A2uiRendererService`
- [`../../core/component-binder.service`](../../core/component-binder.service.ts.md) — `ComponentBinder`
- [`../../core/test-utils`](../../core/test-utils.ts.md) — `setComponentProps`, `createBoundProperty`, `ComponentToProps`

## Exports

없음 (테스트 파일).

## 테스트 케이스 명세

### 픽스처 및 모킹 (첫 번째 `beforeEach` — async)

- `mockRendererService`: `surfaceGroup.getSurface` Jasmine 스파이, 반환값은 빈 `componentsModel` Map, `id: 'mock-catalog'`, 빈 `components` Map의 `catalog` 객체.
- `mockBinder`: `bind` 메서드를 `createSpyObj`로 모킹.
- `TestBed.configureTestingModule`에 `ImageComponent` import, 두 mock provider 등록 후 `compileComponents()` 완료.

### 픽스처 및 초기 프롭 (두 번째 `beforeEach` — sync)

- `TestBed.createComponent(ImageComponent)`로 fixture 생성.
- `surfaceId: 'test-surface'`, `dataContextPath: '/'` input 설정.
- `defaultProps`: `url: createBoundProperty('https://example.com/image.png')`, `fit: createBoundProperty('cover' as const)`, `variant: createBoundProperty('avatar' as const)`.
- `setComponentProps(fixture, defaultProps)` 호출.

### `should create`

- `fixture.detectChanges()` 후 `expect(component).toBeTruthy()` 확인.

### `should render image with url`

- 기본 프롭으로 `detectChanges()` 후 `fixture.nativeElement.querySelector('img')` 조회.
- `img.src`가 truthy인지(URL 바인딩 동작), `img.style.objectFit`이 `'cover'`인지, `img.className`에 `'avatar'`가 포함되는지 검증.

### `should render image with description`

- `description: createBoundProperty('A cute cat')`를 defaultProps에 추가해 `setComponentProps` 재호출 후 `detectChanges()`.
- `img.alt`가 `'A cute cat'`인지 검증.

### `should support all specified variants`

- `['icon', 'avatar', 'smallFeature', 'mediumFeature', 'largeFeature', 'header']` 배열을 `for...of` 루프로 순회.
- 각 variant에 대해 `variant: createBoundProperty(variant)`를 설정하고 `detectChanges()`.
- `img.className`에 해당 variant 문자열이 포함되는지 `toContain(variant)` 검증. 모든 variant 클래스가 `'a2ui-image ' + variant` 형태로 올바르게 적용됨을 확인.

## 동작 흐름

두 개의 `beforeEach` 블록으로 모듈 컴파일(async)과 픽스처 생성(sync)을 분리한다. 각 테스트는 `setComponentProps`로 프롭을 설정하고 `detectChanges()` 후 DOM의 `<img>` 엘리먼트 속성(`src`, `alt`, `style.objectFit`, `className`)을 조회해 검증한다. variant 테스트는 루프로 여러 값을 순차 적용한다.
