# renderers/angular/src/v0_9/catalog/basic/slider.component.spec.ts

## 개요

`SliderComponent`의 Angular 단위 테스트 파일이다. 컴포넌트 생성, 레이블 및 값 렌더링, 그리고 사용자 입력 시 `onUpdate` 콜백 호출 여부를 검증한다.

## 의존성

### 외부 패키지
- `@angular/core/testing`: `ComponentFixture`, `TestBed`
- `@angular/core`: `signal` (alias `angularSignal`)

### 저장소 내부 모듈
- [`./slider.component`](./slider.component.ts.md): 테스트 대상
- [`../../core/a2ui-renderer.service`](../../core/a2ui-renderer.service.ts.md): mock 제공
- [`../../core/component-binder.service`](../../core/component-binder.service.ts.md): mock 제공
- [`../../core/test-utils`](../../core/test-utils.ts.md): `setComponentProps`, `createBoundProperty`, `ComponentToProps`

## 픽스처 및 모킹

### `mockRendererService`
`surfaceGroup.getSurface`가 빈 `componentsModel`(빈 `Map`)과 빈 `catalog`(id `'mock-catalog'`, 빈 `components Map`)를 가진 객체를 반환하는 jasmine spy로 구성된다.

### `mockBinder`
`ComponentBinder`의 `bind` 메서드만 spy로 대체한다(반환값 미설정).

### `defaultProps`
- `label`: `createBoundProperty<string | undefined>('')`
- `min`: `createBoundProperty<number | undefined>(0)`
- `max`: `createBoundProperty(100)`
- `value`: `createBoundProperty(0)`
- `isValid`: `createBoundProperty(true)`
- `validationErrors`: `createBoundProperty<string[]>([])`

`surfaceId`는 `'test-surface'`, `dataContextPath`는 `'/'`로 설정된다.

## 테스트 케이스

### `should create`
`fixture.detectChanges()` 후 `component`가 truthy임을 확인한다.

### `should render slider with value`
`label`을 `'Brightness'`, `value`를 `50`으로 설정한 뒤 `detectChanges()`를 호출한다. `input` 엘리먼트의 `value` 속성이 `'50'`이고, 전체 텍스트 콘텐츠에 `'Brightness'`가 포함됨을 검증한다.

### `should call onUpdate when slider value changes`
`value` 프로퍼티를 `{value: angularSignal(50), raw: 50, onUpdate: onUpdateSpy}` 형태로 설정한다. `input` 엘리먼트의 `value`를 `'75'`로 변경하고 `'input'` 이벤트를 dispatch한 뒤, `onUpdateSpy`가 숫자 `75`로 호출되었는지 검증한다. 이를 통해 `handleInput`에서의 `Number()` 변환과 `onUpdate` 연동을 확인한다.
