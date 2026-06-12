# renderers/angular/src/v0_9/catalog/basic/tabs.component.spec.ts

## 개요

`TabsComponent`의 Angular 단위 테스트 파일이다. 컴포넌트 생성, 탭 렌더링 및 전환, 빈 탭 배열 처리, 그리고 `tabs` 프로퍼티 누락 처리를 검증한다.

## 의존성

### 외부 패키지
- `@angular/core/testing`: `ComponentFixture`, `TestBed`
- `@angular/core`: `Component`, `input`
- `@angular/platform-browser`: `By`
- `@a2ui/web_core/v0_9`: `ComponentModel`, `DynamicString`

### 저장소 내부 모듈
- [`./tabs.component`](./tabs.component.ts.md): 테스트 대상
- [`../../core/a2ui-renderer.service`](../../core/a2ui-renderer.service.ts.md): mock 제공
- [`../../core/component-binder.service`](../../core/component-binder.service.ts.md): mock 제공
- [`../../core/test-utils`](../../core/test-utils.ts.md): `setComponentProps`, `createBoundProperty`, `ComponentToProps`

## 픽스처 및 모킹

### `DummyTextComponent`
`selector: 'dummy-text-for-tabs'`, `template: '<div>{{text}}</div>'`인 standalone 컴포넌트. `text?: string`, `props`, `surfaceId`, `componentId`, `dataContextPath`를 `input()`으로 선언한다.

### `mockRendererService`
`surfaceGroup.getSurface`가 다음 객체를 반환한다.
- `componentsModel`: `'content-1'`과 `'content-2'` 두 `ComponentModel` 인스턴스가 담긴 `Map`. `content-1`의 properties는 `{text: {value: 'Content 1'}}`, `content-2`는 `{text: {value: 'Content 2'}}`.
- `catalog.components`: `'Text'` 타입이 `{type: 'Text', component: DummyTextComponent}`로 등록된 `Map`.

### `mockBinder`
`ComponentBinder`의 `bind` spy (반환값 미설정).

### `defaultProps`
- `tabs`: `createBoundProperty<{title: DynamicString; child: string}[]>([])`

`surfaceId`는 `'test-surface'`, `dataContextPath`는 `'/'`로 설정된다.

## 테스트 케이스

### `should create`
`fixture.detectChanges()` 후 `component`가 truthy임을 확인한다.

### `should render tabs and switch content`
`tabs`를 `[{title:'Tab 1', child:'content-1'}, {title:'Tab 2', child:'content-2'}]`로 설정한 뒤 `detectChanges()`를 호출한다.
- `.a2ui-tab-button` 요소가 2개이고 첫 번째 텍스트가 `'Tab 1'`을 포함함을 검증한다.
- `By.css('a2ui-v09-component-host')` 쿼리로 첫 번째 host의 `componentKey()`가 `{id:'content-1', basePath:'/'}`임을 확인한다.
- 두 번째 탭 버튼을 `click()`하고 `detectChanges()` 후 host의 `componentKey()`가 `{id:'content-2', basePath:'/'}`로 변경됨을 검증한다. 이를 통해 `setActiveTab` 및 `normalizedActiveTabChild` 정규화 로직을 통합 검증한다.

### `should handle missing tabs property`
`defaultProps`(빈 탭 배열)로 `detectChanges()`를 호출한다. `component.tabs()`가 `[]`이고, `.a2ui-tab-button`이 0개임을 검증한다.

### `should handle empty tabs array`
명시적으로 빈 배열 `[]`을 `tabs`에 설정한 뒤 동일하게 `component.tabs()`가 `[]`이고 버튼이 없음을 검증한다.
