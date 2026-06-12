# renderers/angular/src/v0_9/catalog/basic/icon.component.spec.ts

## 개요

`IconComponent`에 대한 Angular 단위 테스트 파일이다. 컴포넌트 생성, 문자열 이름 기반 아이콘 렌더링, camelCase→snake_case 자동 변환, `ICON_NAME_OVERRIDES` 특수 이름 매핑, SVG 경로 객체 입력 시 `<svg>` 렌더링 분기 동작을 검증한다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture`, `TestBed`

### 저장소 내부 모듈
- [`./icon.component`](./icon.component.ts.md) — `IconComponent`
- [`../../core/a2ui-renderer.service`](../../core/a2ui-renderer.service.ts.md) — `A2uiRendererService`
- [`../../core/component-binder.service`](../../core/component-binder.service.ts.md) — `ComponentBinder`
- [`../../core/test-utils`](../../core/test-utils.ts.md) — `setComponentProps`, `createBoundProperty`, `ComponentToProps`

## Exports

없음 (테스트 파일).

## 테스트 케이스 명세

### 픽스처 및 모킹 (첫 번째 `beforeEach` — async)

- `mockRendererService`: `surfaceGroup.getSurface` Jasmine 스파이, 반환값은 빈 `componentsModel` Map, `id: 'mock-catalog'`, 빈 `components` Map의 `catalog` 객체.
- `mockBinder`: `bind` 메서드를 `createSpyObj`로 모킹.
- `TestBed.configureTestingModule`에 `IconComponent` import, 두 mock provider 등록 후 `compileComponents()` 완료.

### 픽스처 및 초기 프롭 (두 번째 `beforeEach` — sync)

- `TestBed.createComponent(IconComponent)`로 fixture 생성.
- `surfaceId: 'test-surface'`, `dataContextPath: '/'` input 설정.
- `defaultProps`: `name: createBoundProperty('search' as const)`.
- `setComponentProps(fixture, defaultProps)` 호출.

### `should create`

- `fixture.detectChanges()` 후 `expect(component).toBeTruthy()` 확인.

### `should render named icon`

- 기본 프롭 `name: 'search'`로 `detectChanges()` 후 `fixture.nativeElement.querySelector('.a2ui-icon')` 조회.
- `icon.textContent.trim()`이 `'search'`인지 검증. 소문자, 단일 단어 이름은 변환 없이 그대로 렌더링됨을 확인.

### `should convert camelCase icon names to snake_case`

- `name: createBoundProperty('shoppingCart' as const)`로 재설정 후 `detectChanges()`.
- `.a2ui-icon` 텍스트가 `'shopping_cart'`인지 검증. camelCase → snake_case 변환 로직(정규식 `/[A-Z]/g`)이 적용됨을 확인.

### `should map "play" to "play_arrow"`

- `name: createBoundProperty('play' as const)`로 설정 후 `detectChanges()`.
- `.a2ui-icon` 텍스트가 `'play_arrow'`인지 검증. `ICON_NAME_OVERRIDES['play']` 매핑이 우선 적용됨을 확인.

### `should map "rewind" to "fast_rewind"`

- `name: createBoundProperty('rewind' as const)`로 설정 후 `detectChanges()`.
- `.a2ui-icon` 텍스트가 `'fast_rewind'`인지 검증. `ICON_NAME_OVERRIDES['rewind']` 매핑 확인.

### `should map "favoriteOff" to "favorite_border"`

- `name: createBoundProperty('favoriteOff' as const)`로 설정 후 `detectChanges()`.
- `.a2ui-icon` 텍스트가 `'favorite_border'`인지 검증. `ICON_NAME_OVERRIDES['favoriteOff']` 매핑 확인.

### `should map "starOff" to "star_border"`

- `name: createBoundProperty('starOff' as const)`로 설정 후 `detectChanges()`.
- `.a2ui-icon` 텍스트가 `'star_border'`인지 검증. `ICON_NAME_OVERRIDES['starOff']` 매핑 확인.

### `should render path icon`

- `name`을 `createBoundProperty({ svgPath: 'M10 10...' })`로 설정 (타입 캐스팅 `as unknown as ComponentToProps<IconComponent>['name']` 사용) 후 `detectChanges()`.
- `fixture.nativeElement.querySelector('svg')`가 truthy인지 검증.
- 이름이 `{ svgPath: string }` 객체 형태일 때 `<svg>` 렌더링 분기(`isSvgPath()`)가 동작함을 확인.

## 동작 흐름

두 개의 `beforeEach` 블록으로 모듈 컴파일(async)과 픽스처 생성(sync)을 분리한다. 각 테스트는 `name` 프롭 하나만 바꿔가며 `.a2ui-icon` 엘리먼트의 `textContent` 또는 `svg` 엘리먼트 존재 여부로 아이콘 렌더링 결과를 검증한다.
