# renderers/angular/src/v0_9/catalog/basic/divider.component.spec.ts

## 개요

`DividerComponent`에 대한 Angular 단위 테스트 파일이다. Jasmine/TestBed를 사용해 컴포넌트 생성과 수평/수직 구분선 렌더링 동작을 검증한다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture`, `TestBed`

### 저장소 내부 모듈
- [`./divider.component`](./divider.component.ts.md) — `DividerComponent`
- [`../../core/a2ui-renderer.service`](../../core/a2ui-renderer.service.ts.md) — `A2uiRendererService`
- [`../../core/component-binder.service`](../../core/component-binder.service.ts.md) — `ComponentBinder`
- [`../../core/test-utils`](../../core/test-utils.ts.md) — `setComponentProps`, `createBoundProperty`, `ComponentToProps`

## Exports

없음 (테스트 파일).

## 테스트 케이스 명세

### 픽스처 및 모킹 (첫 번째 `beforeEach` — async)

- `mockRendererService`: `surfaceGroup.getSurface` Jasmine 스파이, 반환값은 빈 `componentsModel` Map, `id: 'mock-catalog'`, 빈 `components` Map을 가진 `catalog` 객체.
- `mockBinder`: `ComponentBinder`의 `bind` 메서드를 `createSpyObj`로 모킹.
- `TestBed.configureTestingModule`에 `DividerComponent`를 imports, 두 mock provider를 등록하고 `compileComponents()` 완료.

### 픽스처 및 초기 프롭 (두 번째 `beforeEach` — sync)

- `TestBed.createComponent(DividerComponent)`로 fixture 생성, `component` 할당.
- `fixture.componentRef.setInput('surfaceId', 'test-surface')`, `setInput('dataContextPath', '/')` 설정.
- `defaultProps`: `axis: createBoundProperty('horizontal' as const)`.
- `setComponentProps(fixture, defaultProps)` 호출.

### `should create`

- `fixture.detectChanges()` 후 `expect(component).toBeTruthy()` 확인.
- 기본 프롭으로 컴포넌트가 정상 생성되는지 검증.

### `should render horizontal divider by default`

- `fixture.detectChanges()` 후 `fixture.nativeElement.querySelector('.a2ui-divider')`로 divider 엘리먼트 조회.
- `divider.classList`에 `'horizontal'`이 포함되어 있는지 `toContain('horizontal')` 검증.
- 기본 `axis` 값 `'horizontal'`이 올바른 CSS 클래스로 반영됨을 확인.

### `should render vertical divider`

- `setComponentProps(fixture, { ...defaultProps, axis: createBoundProperty('vertical' as const) })`로 프롭 재설정.
- `fixture.detectChanges()` 후 `.a2ui-divider` 엘리먼트의 classList에 `'vertical'`이 포함되는지 검증.

## 동작 흐름

두 개의 `beforeEach` 블록으로 모듈 컴파일(async)과 픽스처 생성(sync)을 분리한다. 각 테스트는 `setComponentProps`로 필요한 `axis` 값을 설정하고, `fixture.detectChanges()` 이후 `.a2ui-divider` 엘리먼트의 classList를 조회해 방향 클래스 적용 여부를 검증한다.
