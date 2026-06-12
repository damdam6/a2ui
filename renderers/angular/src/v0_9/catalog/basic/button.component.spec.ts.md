# renderers/angular/src/v0_9/catalog/basic/button.component.spec.ts

## 개요

`ButtonComponent`의 Angular 단위 테스트 파일이다. Jasmine + Angular `TestBed`를 사용하여 버튼 렌더링, 타입 설정, 클릭 액션 디스패치, isValid 비활성화, 테마 색상 반영 동작을 검증한다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture`, `TestBed`
- `@angular/core` — `Component`, `input`
- `@angular/platform-browser` — `By`
- `@a2ui/web_core/v0_9` — `Action`, `ComponentModel`

### 저장소 내부 모듈
- [`./button.component`](./button.component.ts.md)
- [`../../core/a2ui-renderer.service`](../../core/a2ui-renderer.service.ts.md)
- [`../../core/component-binder.service`](../../core/component-binder.service.ts.md)
- [`../../core/test-utils`](../../core/test-utils.ts.md)

## 픽스처 / 모킹

- **`mockSurface`**: `dispatchAction` 스파이, `componentsModel`(Map에 `child1` ComponentModel 포함), `catalog`(인라인 정의된 `DummyText` 스탠드얼론 컴포넌트 포함)를 가진 객체. `DummyText`는 `props`, `surfaceId`, `componentId`, `dataContextPath` input을 가지고 `'Dummy Text'` 텍스트를 렌더링.
- **`mockSurfaceGroup`**: `getSurface` 스파이(항상 `mockSurface` 반환).
- **`mockRendererService`**: `surfaceGroup: mockSurfaceGroup` 구조.
- **`mockBinder`**: `bind` 메서드 스파이, `{text: {value: () => 'bound text'}}` 반환.
- **`defaultProps`**: `variant='primary'`, `child={id:'child1', basePath:'/'}`, `action={event:{name:'test-action'}}`, `isValid=true`, `validationErrors=[]` 로 구성된 기본 프로퍼티 맵.
- TestBed에 `A2uiRendererService`와 `ComponentBinder`를 목(mock)으로 제공. `surfaceId='surf1'`, `componentId='comp1'`.

## 테스트 케이스

| 테스트 이름 | 검증 동작 |
|---|---|
| `should create` | 컴포넌트 인스턴스가 truthy인지 확인 |
| `should set button type to submit for primary variant` | `variant='primary'`일 때 `<button>` 엘리먼트의 `type` 속성이 `'submit'`인지 확인 |
| `should set button type to button for non-primary variant` | `variant='default'`일 때 `<button>`의 `type`이 `'button'`인지 확인 |
| `should apply variant class` | `variant='primary'`일 때 버튼 엘리먼트의 classList에 `'primary'`가 포함되는지 확인 |
| `should handle click and dispatch action with sourceComponentId` | 버튼 클릭 시 `getSurface`가 `'surf1'`으로 호출되고, `dispatchAction`이 임의의 객체와 `'comp1'`으로 호출되는지 확인 |
| `should show child component host if child prop is present` | `child` prop이 있을 때 `a2ui-v09-component-host` 엘리먼트가 존재하고 `componentKey()`가 `{id:'child1', basePath:'/'}` 인지 확인 |
| `should be disabled when isValid is false` | `isValid` 시그널 값을 `true`→`false`로 변경 후 버튼의 `disabled` 속성이 `true`가 되는지 확인 |
| `should override the button default background color when primary color is set` | `mockSurface.theme.primaryColor = 'red'` 설정 후 버튼의 computed 배경색이 `'rgb(255, 0, 0)'`인지 확인 |
