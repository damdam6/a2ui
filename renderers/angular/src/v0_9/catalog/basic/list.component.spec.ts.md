# renderers/angular/src/v0_9/catalog/basic/list.component.spec.ts

## 개요

`ListComponent`에 대한 Angular 단위 테스트 파일이다. 컴포넌트 생성, 자식 컴포넌트 렌더링, ordered/unordered 목록 HTML 태그 적용, 비목록 스타일 시 `<div>` 폴백, 수평 방향 CSS 클래스 적용 동작을 검증한다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture`, `TestBed`
- `@angular/core` — `Component`, `input`
- `@a2ui/web_core/v0_9` — `ComponentModel`

### 저장소 내부 모듈
- [`./list.component`](./list.component.ts.md) — `ListComponent`
- [`../../core/a2ui-renderer.service`](../../core/a2ui-renderer.service.ts.md) — `A2uiRendererService`
- [`../../core/component-binder.service`](../../core/component-binder.service.ts.md) — `ComponentBinder`, `Child`
- [`../../core/test-utils`](../../core/test-utils.ts.md) — `setComponentProps`, `createBoundProperty`, `ComponentToProps`

## Exports

없음 (테스트 파일).

## 테스트 케이스 명세

### 더미 컴포넌트 (`DummyTextComponent`)

`selector: 'dummy-text-for-list'`, standalone 컴포넌트. 템플릿 `<div>{{text}}</div>`. `text?: string`, `props`, `surfaceId`, `componentId`, `dataContextPath`를 `input()`으로 선언한 최소 컴포넌트. `mockRendererService`의 `catalog.components` Map에 `'Text'` 키로 등록되어 `ComponentHostComponent`의 동적 컴포넌트 해석에 사용된다.

### 픽스처 및 모킹 (`beforeEach` — async)

- `mockRendererService.surfaceGroup.getSurface` 반환값:
  - `componentsModel` Map: `'child-1'` → `new ComponentModel('child-1', 'Text', {text: {value: 'Child 1'}})`, `'child-2'` → `new ComponentModel('child-2', 'Text', {text: {value: 'Child 2'}})`.
  - `catalog`: `id: 'mock-catalog'`, `components` Map에 `'Text': {type: 'Text', component: DummyTextComponent}`.
- `mockBinder`: `bind` 메서드를 `createSpyObj`로 모킹.
- `TestBed.configureTestingModule`에 `ListComponent` import, 두 mock provider 등록 후 `compileComponents()`.
- `fixture = TestBed.createComponent(ListComponent)`, `surfaceId: 'test-surface'`, `dataContextPath: '/'` 설정.
- `defaultProps`: `children: createBoundProperty<Child[]>([])`, `direction: createBoundProperty<'vertical' | 'horizontal' | undefined>('vertical')`, `listStyle: createBoundProperty<'none' | 'ordered' | 'unordered' | undefined>('none')`.
- `setComponentProps(fixture, defaultProps)` 호출.

### `should create`

- `fixture.detectChanges()` 후 `expect(component).toBeTruthy()`.

### `should render children`

- `children: createBoundProperty([{id: 'child-1', basePath: '/'}, {id: 'child-2', basePath: '/'}])`으로 재설정 후 `detectChanges()`.
- `fixture.nativeElement.querySelectorAll('a2ui-v09-component-host')`로 호스트 엘리먼트 수가 2인지 검증.

### `should render as ordered list`

- `children: [{id: 'child-1', basePath: '/'}]`, `listStyle: createBoundProperty('ordered')` 설정 후 `detectChanges()`.
- `fixture.nativeElement.querySelector('ol')`이 truthy인지 검증. `listStyle === 'ordered'` 시 `<ol>` 태그 사용 확인.

### `should render as unordered list`

- `children: [{id: 'child-1', basePath: '/'}]`, `listStyle: createBoundProperty('unordered')` 설정 후 `detectChanges()`.
- `fixture.nativeElement.querySelector('ul')`이 truthy인지 검증.

### `should render fallback list when style is not list style`

- `children: [{id: 'child-1', basePath: '/'}]`, `listStyle: createBoundProperty('div' as 'none' | 'ordered' | 'unordered' | undefined)` (강제 타입 캐스팅)으로 설정 후 `detectChanges()`.
- `fixture.nativeElement.querySelector('.a2ui-list')` 엘리먼트의 `tagName.toLowerCase()`가 `'div'`인지 검증. `'ordered'`/`'unordered'` 외 값은 `<div>` 폴백임을 확인.

### `should apply horizontal orientation class`

- `children: [{id: 'child-1', basePath: '/'}]`, `direction: createBoundProperty<'vertical' | 'horizontal' | undefined>('horizontal')` 설정 후 `detectChanges()`.
- `fixture.nativeElement.querySelector('.a2ui-list').classList`에 `'horizontal'`이 포함되는지 검증.

## 동작 흐름

단일 `beforeEach`(async) 블록으로 모듈 컴파일과 픽스처 생성을 모두 처리한다. 각 테스트는 `setComponentProps`로 `children`, `direction`, `listStyle` 프롭을 조합해 설정하고, DOM 쿼리로 렌더링된 HTML 태그 종류, 엘리먼트 수, CSS 클래스를 확인해 동작을 검증한다.
