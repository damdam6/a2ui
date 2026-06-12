# renderers/angular/src/v0_9/catalog/basic/modal.component.spec.ts

## 개요

`ModalComponent`에 대한 Angular 단위 테스트 파일이다. 컴포넌트 생성, 트리거 클릭 시 모달 열기 및 콘텐츠 렌더링, 닫기 버튼 클릭 시 모달 닫기, 오버레이 배경 클릭 시 모달 닫기, trigger/content 프롭 미설정 시 안전 렌더링(컴포넌트 호스트 없음) 동작을 검증한다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture`, `TestBed`
- `@angular/core` — `Component`, `input`
- `@angular/platform-browser` — `By`
- `@a2ui/web_core/v0_9` — `ComponentModel`

### 저장소 내부 모듈
- [`./modal.component`](./modal.component.ts.md) — `ModalComponent`
- [`../../core/a2ui-renderer.service`](../../core/a2ui-renderer.service.ts.md) — `A2uiRendererService`
- [`../../core/component-binder.service`](../../core/component-binder.service.ts.md) — `ComponentBinder`
- [`../../core/test-utils`](../../core/test-utils.ts.md) — `setComponentProps`, `createBoundProperty`, `ComponentToProps`

## Exports

없음 (테스트 파일).

## 테스트 케이스 명세

### 더미 컴포넌트 (`DummyTextComponent`)

`selector: 'dummy-text-for-modal'`, standalone 컴포넌트. 템플릿 `<div>{{text}}</div>`. `text?: string`, `props`, `surfaceId`, `componentId`, `dataContextPath`를 `input()`으로 선언. `mockRendererService`의 `catalog.components`에 `'Text'` 키로 등록된다.

### 픽스처 및 모킹 (`beforeEach` — async)

- `mockRendererService.surfaceGroup.getSurface` 반환값:
  - `componentsModel` Map: `'trigger-btn'` → `new ComponentModel('trigger-btn', 'Text', {text: {value: 'Open'}})`, `'modal-content'` → `new ComponentModel('modal-content', 'Text', {text: {value: 'Modal'}})`.
  - `catalog`: `id: 'mock-catalog'`, `components` Map에 `'Text': {type: 'Text', component: DummyTextComponent}`.
- `mockBinder`: `bind` 메서드를 `createSpyObj`로 모킹.
- `TestBed.configureTestingModule`에 `ModalComponent` import, 두 mock provider 등록 후 `compileComponents()`.
- `fixture = TestBed.createComponent(ModalComponent)`, `surfaceId: 'test-surface'`, `dataContextPath: '/'` 설정.
- `defaultProps`: 빈 객체 `{} as ComponentToProps<ModalComponent>` (trigger, content 없음).
- `setComponentProps(fixture, defaultProps)` 호출.

### `should create`

- `fixture.detectChanges()` 후 `expect(component).toBeTruthy()`.

### `should render trigger and open modal on click`

- `trigger: createBoundProperty({id: 'trigger-btn', basePath: '/'})`, `content: createBoundProperty({id: 'modal-content', basePath: '/'})` 설정 후 `detectChanges()`.
- `By.css('.a2ui-modal-trigger a2ui-v09-component-host')`로 트리거 내부 컴포넌트 호스트 인스턴스 조회 → `componentKey()`가 `{id: 'trigger-btn', basePath: '/'}` 임을 확인.
- 초기 상태에서 `querySelector('.a2ui-modal-overlay')`가 falsy(모달 닫힘)임을 확인.
- `.a2ui-modal-trigger.click()` 후 `detectChanges()` → `.a2ui-modal-overlay`가 DOM에 존재하는지 확인(모달 열림).
- `By.css('.a2ui-modal-overlay a2ui-v09-component-host')`로 콘텐츠 호스트 인스턴스 조회 → `componentKey()`가 `{id: 'modal-content', basePath: '/'}` 임을 검증.

### `should close modal when close button clicked`

- trigger, content 프롭 설정 후 `.a2ui-modal-trigger` 클릭으로 모달 열기.
- `detectChanges()` 후 `.a2ui-modal-overlay` 존재 확인.
- `.a2ui-modal-close` 버튼 클릭 후 `detectChanges()` → `.a2ui-modal-overlay`가 DOM에서 사라졌는지 검증.

### `should close modal when overlay clicked`

- trigger, content 프롭 설정 후 `.a2ui-modal-trigger` 클릭으로 모달 열기.
- `detectChanges()` 후 `.a2ui-modal-overlay` 존재 확인.
- `.a2ui-modal-overlay` 자체를 클릭 후 `detectChanges()` → `.a2ui-modal-overlay`가 DOM에서 사라졌는지 검증. 오버레이 배경 클릭으로 닫히는 동작 확인.

### `should handle missing trigger or content`

- `defaultProps` (빈 객체)로 `detectChanges()`.
- `component.trigger()`가 `undefined`인지, `component.content()`가 `undefined`인지 확인.
- `querySelector('a2ui-v09-component-host')`가 falsy인지 검증. trigger/content 프롭이 없을 때 컴포넌트 호스트를 렌더링하지 않고 안전하게 처리됨을 확인.

## 동작 흐름

단일 `beforeEach`(async) 블록으로 모듈 컴파일과 픽스처 초기화를 모두 처리한다. 모달 열기/닫기 테스트는 "클릭 → `detectChanges()` → DOM 조회" 패턴을 따른다. `By.css`를 사용한 Angular 디버그 엘리먼트 조회(컴포넌트 인스턴스 프로퍼티 접근)와 직접 `nativeElement` DOM 쿼리(엘리먼트 존재 여부 확인)를 혼용해 컴포넌트 상태와 DOM 상태를 함께 검증한다.
