# renderers/angular/src/v0_9/core/component-host.component.spec.ts

## 개요

`ComponentHostComponent`의 통합 단위 테스트 파일이다. 카탈로그로부터 컴포넌트 타입 해석, props 바인딩, `dataContextPath` 전파, 서피스/컴포넌트 미발견 경고, 컴포넌트 모델 업데이트 반응성, 그리고 소멸 처리를 검증한다.

## 의존성

### 외부 패키지
- `@angular/core/testing`: `TestBed`, `ComponentFixture`
- `@angular/platform-browser`: `By`
- `@angular/core/testing`: `Component`, `Input` (테스트 더블 정의용)
- `@a2ui/web_core/v0_9`: `ComponentModel`, `SurfaceComponentsModel`, `SurfaceModel`

### 저장소 내부 모듈
- [`./component-host.component`](./component-host.component.ts.md)
- [`./a2ui-renderer.service`](./a2ui-renderer.service.ts.md)

## Exports

없음 (테스트 파일).

## 테스트 케이스 명세

### 픽스처 및 모킹 설정

**테스트 더블 컴포넌트 `TestChildComponent`**:
- `@Component({ selector: 'test-child', template: '<div>Child Component</div>' })`
- 입력: `@Input() props`, `@Input() surfaceId`, `@Input() componentId`, `@Input() dataContextPath`

**`beforeEach` (async)**:
- `mockCatalog`: `id: 'test-catalog'`, `components: new Map([['TestType', { component: TestChildComponent }]])`
- `mockSurfaceComponentsModel`: `new SurfaceComponentsModel()`에 `new ComponentModel('comp1', 'TestType', { text: 'Hello' })` 추가.
- `mockSurface`: `{ componentsModel: mockSurfaceComponentsModel, catalog: mockCatalog }`
- `mockSurfaceGroup`: `getSurface` spy가 `mockSurface`를 반환.
- `mockRendererService`: `{ surfaceGroup: mockSurfaceGroup }`
- `TestBed`에 `ComponentHostComponent` import, `A2uiRendererService` 목 제공.
- `componentKey: { id: 'comp1', basePath: '/' }`, `surfaceId: 'surf1'`로 초기 입력 설정.

---

### `should be created`

컴포넌트 인스턴스가 truthy임을 확인.

---

### `Component Initialization` — `should resolve component type and bind props`

**검증 동작**: `detectChanges()` 후 DOM에 `TestChildComponent`가 렌더링되고, `props.text.value()` = `'Hello'`, `surfaceId` = `'surf1'`, `componentId` = `'comp1'`, `dataContextPath` = `'/'`임을 확인. `mockSurfaceGroup.getSurface`가 `'surf1'`로 호출됨을 확인.

---

### `Component Initialization` — `should use provided dataContextPath for ComponentContext`

**검증 동작**: `componentKey`를 `{ id: 'comp1', basePath: '/nested/path' }`로 변경 후, 자식 컴포넌트의 `dataContextPath`가 `'/nested/path'`임을 확인.

---

### `Component Initialization` — `should update props when component model is updated`

**검증 동작**: 초기 렌더 후 `compModel.properties = { text: 'Hello', newProp: 'new value' }`를 설정하고, 두 번째 `detectChanges()` 후 `props.newProp.value()` = `'new value'`임을 확인하여 동적 프로퍼티 추가 반응성을 검증.

---

### `Component Initialization` — `should warn and return if surface not found`

**검증 동작**: `mockSurfaceGroup.getSurface.and.returnValue(null)` 설정 후 `detectChanges()`. `TestChildComponent`가 렌더링되지 않고, `console.warn`이 `'Surface surf1 not found'`로 호출됨을 확인.

---

### `Component Initialization` — `should warn and return if component model not found`

**검증 동작**: `mockSurface.componentsModel.dispose()` 후 `detectChanges()`. `TestChildComponent`가 렌더링되지 않고, `console.warn`이 `'Component comp1 not found in surface surf1. Waiting for it...'`로 호출됨을 확인.

---

### `Component Initialization` — `should error and return if component type not in catalog`

**검증 동작**: `mockCatalog.components.clear()` 후 `detectChanges()`. `TestChildComponent`가 렌더링되지 않고, `console.error`가 `'Component type "TestType" not found in catalog "test-catalog"'`로 호출됨을 확인.

---

### `Component Initialization` — `should trigger destroyRef on destroy`

**검증 동작**: `detectChanges()` 후 `fixture.destroy()` 호출 시 충돌 없이 컴포넌트가 소멸됨을 확인.

---

### `Template rendering` — `should render the resolved component`

**검증 동작**: `detectChanges()` 후 `fixture.nativeElement.innerHTML`에 `'Child Component'` 텍스트가 포함됨을 확인.

---

### `Template rendering` — `should pass dataContextPath to the rendered component`

**검증 동작**: `componentKey`를 `{ id: 'comp1', basePath: '/some/path' }`로 설정하고, 자식의 `dataContextPath`가 `'/some/path'`임을 확인.

## 동작 흐름

테스트는 `TestBed`에서 실제 `ComponentHostComponent`를 마운트하되 `A2uiRendererService`를 목으로 교체한다. `fixture.detectChanges()`가 Angular의 변경 감지 사이클을 트리거하여 `effect()`와 `setupComponent()`를 실행시킨다. DOM 조회(`By.directive`)와 컴포넌트 인스턴스 프로퍼티 검사를 병행하여 렌더링 정확성을 검증한다.
