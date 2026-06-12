# renderers/angular/src/v0_9/catalog/basic/column.component.spec.ts

## 개요

`ColumnComponent`의 Angular 단위 테스트 파일이다. Jasmine + `TestBed`를 사용하여 컬럼의 생성, flex 레이아웃 스타일 적용(justify/align), weight 프로퍼티에 따른 flex 스타일, 일반 자식 및 반복(repeating) 자식 렌더링, missing 프로퍼티 처리를 검증한다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture`, `TestBed`
- `@angular/core` — `Component`, `input`, `signal`
- `@angular/platform-browser` — `By`
- `@a2ui/web_core/v0_9` — `ComponentModel`

### 저장소 내부 모듈
- [`./column.component`](./column.component.ts.md)
- [`../../core/a2ui-renderer.service`](../../core/a2ui-renderer.service.ts.md)
- [`../../core/component-binder.service`](../../core/component-binder.service.ts.md)
- [`../../core/test-utils`](../../core/test-utils.ts.md)

## 픽스처 / 모킹

- **`DummyChild`**: `selector: 'dummy-child'`, 템플릿 `'Dummy Child'`. `props`, `surfaceId`, `componentId`, `dataContextPath` input을 가지는 스탠드얼론 컴포넌트. 카탈로그의 `'Child'` 타입 자리에 사용된다.
- **`mockSurface`**: `componentsModel`(Map에 `child1`, `child2`, `template1` ComponentModel 포함), `catalog`(`DummyChild` 컴포넌트를 `'Child'` 키로 등록).
- **`mockSurfaceGroup`**: `getSurface` 스파이가 `mockSurface` 반환.
- **`mockRendererService`**: `surfaceGroup: mockSurfaceGroup`.
- **`mockBinder`**: `bind` 스파이, `{text: {value: () => 'bound'}}` 반환.
- **`defaultProps`**: `justify='start'`, `align='stretch'`, `children=[{id:'child1', basePath:'/'}, {id:'child2', basePath:'/'}]`.
- `surfaceId='surf1'`로 설정.

## 테스트 케이스

| 테스트 이름 | 검증 동작 |
|---|---|
| `should create` | 컴포넌트 인스턴스가 truthy인지 확인 |
| `should apply flex styles from props` | `justify='start'`, `align='stretch'` 설정 후 호스트 엘리먼트의 computed style에서 `justifyContent`가 `'flex-start'`, `alignItems`가 `'stretch'`인지 확인 |
| `should apply flex style from weight prop` | `weight=2` 설정 후 computed style의 `flex`가 `'2 1 0%'`인지 확인 |
| `should apply flex style from weight prop when value is 0` | `weight=0` 설정 후 computed style의 `flex`가 `'0 1 0%'`인지 확인 (0이 null과 다르게 처리됨을 검증) |
| `should not apply flex style when weight prop is null` | `weight=undefined` 설정 후 computed style의 `flex`가 `'2 1 0%'`나 `'0 1 0%'`가 아닌지 확인 |
| `should render non-repeating children` | `children=[{id:'child1',...}, {id:'child2',...}]`일 때 `a2ui-v09-component-host` 2개가 렌더링되고 각각 올바른 `componentKey()`를 가지는지 확인 |
| `should render repeating children` | `children.value`가 `template1` 기반의 두 반복 항목(`/items/0`, `/items/1`)을 가진 시그널이고 `raw`와 `template`에 `componentId: 'template1'`이 설정된 경우, `a2ui-v09-component-host` 2개가 각각 올바른 `componentKey()`(`basePath: '/items/0'`, `'/items/1'`)를 가지는지 확인 |
| `should handle missing justify and align properties` | `justify`/`align` 없이 `children`만 설정 시 호스트 엘리먼트의 `justify-content`와 `align-items` 스타일이 falsy인지 확인 |
