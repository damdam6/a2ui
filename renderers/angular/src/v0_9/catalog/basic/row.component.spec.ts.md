# renderers/angular/src/v0_9/catalog/basic/row.component.spec.ts

## 개요

`RowComponent`의 Angular 단위 테스트 파일이다. Jasmine/TestBed 환경에서 flex 스타일 바인딩, 정적 자식 렌더링, 반복 자식 렌더링, 그리고 선택적 프로퍼티 누락 처리를 검증한다.

## 의존성

### 외부 패키지
- `@angular/core/testing`: `ComponentFixture`, `TestBed`
- `@angular/core`: `Component`, `input`, `signal`
- `@angular/platform-browser`: `By`
- `@a2ui/web_core/v0_9`: `ComponentModel`

### 저장소 내부 모듈
- [`./row.component`](./row.component.ts.md): 테스트 대상
- [`../../core/a2ui-renderer.service`](../../core/a2ui-renderer.service.ts.md): mock 제공
- [`../../core/component-binder.service`](../../core/component-binder.service.ts.md): mock 제공 (`ComponentBinder`)
- [`../../core/test-utils`](../../core/test-utils.ts.md): `setComponentProps`, `createBoundProperty`, `ComponentToProps`

## 픽스처 및 모킹

### `DummyChild` 컴포넌트
`selector: 'dummy-child'`, `template: 'Dummy Child'`인 standalone 인라인 컴포넌트. `props`, `surfaceId`, `componentId`, `dataContextPath`를 `input()`으로 선언한다.

### `mockSurface`
`componentsModel`: `child1`, `child2`, `template1` 세 개의 `ComponentModel` 인스턴스가 담긴 `Map`. `catalog.components`에는 `'Child'` 타입이 `{component: DummyChild}`로 등록된다.

### `mockSurfaceGroup`
`getSurface` jasmine spy가 항상 `mockSurface`를 반환한다.

### `mockRendererService`
`{surfaceGroup: mockSurfaceGroup}` 형태로 `A2uiRendererService`를 대체한다.

### `mockBinder`
`ComponentBinder`의 `bind` 메서드를 spy로 대체하며, `{text: {value: () => 'bound'}}`를 반환한다.

### `defaultProps`
- `justify`: `createBoundProperty('center')`
- `align`: `createBoundProperty('stretch')`
- `children`: `createBoundProperty([{id:'child1',basePath:'/'},{id:'child2',basePath:'/'}])`

## 테스트 케이스

### `should create`
`fixture.detectChanges()` 후 `component`가 truthy임을 검증한다.

### `should apply flex styles from props`
`fixture.detectChanges()` 후 `window.getComputedStyle`로 호스트 엘리먼트의 `justifyContent`가 `'center'`, `alignItems`가 `'stretch'`임을 검증한다. `JUSTIFY_MAP`과 `ALIGN_MAP`을 통한 CSS 변환이 올바르게 적용되었는지 확인한다.

### `should render non-repeating children`
`fixture.detectChanges()` 후 CSS 셀렉터 `'a2ui-v09-component-host'`로 쿼리한 결과가 2개임을 확인하고, 첫 번째의 `componentKey()`가 `{id:'child1',basePath:'/'}`, 두 번째가 `{id:'child2',basePath:'/'}`임을 검증한다.

### `should render repeating children`
`children` 프로퍼티를 `{id:'template1', basePath:'/items/0'}`, `{id:'template1', basePath:'/items/1'}` 두 항목으로 설정하고, `raw.componentId`가 `'template1'`, `raw.path`가 `'items'`, `template`이 `{id:'template1', path:'items'}`로 구성된 반복 템플릿 형태로 지정한다. 렌더링 후 2개의 host가 존재하고 각 `componentKey()`가 해당 `basePath`를 가짐을 검증한다.

### `should handle missing justify and align properties`
`justify`와 `align` 없이 `children`만 설정한 뒤 `detectChanges()`를 호출한다. 호스트 엘리먼트의 인라인 `styles['justify-content']`와 `styles['align-items']`가 falsy임을 검증하여, 선택적 레이아웃 프로퍼티 누락 시 스타일이 설정되지 않음을 확인한다.
