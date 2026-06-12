# renderers/angular/src/v0_9/core/surface.component.spec.ts

## 개요

`SurfaceComponent`의 단위 테스트 파일로, 컴포넌트 생성, 입력 바인딩 전파, 기본값 동작을 검증한다. `A2uiRendererService`와 `ComponentBinder`를 목 객체로 대체하여 `SurfaceComponent`가 내부 `ComponentHostComponent`에 올바른 입력값을 전달하는지 확인한다. 테스트용 인라인 컴포넌트 `TestTextComponent`도 정의한다.

## 의존성

### 외부 패키지
- `@angular/core` — `Component`, `Input`, `ChangeDetectionStrategy`
- `@angular/core/testing` — `ComponentFixture`, `TestBed`
- `@angular/platform-browser` — `By`
- `@a2ui/web_core/v0_9` — `ComponentModel`

### 저장소 내부 모듈
- [`./surface.component`](./surface.component.ts.md) — `SurfaceComponent`
- [`./component-host.component`](./component-host.component.ts.md) — `ComponentHostComponent`
- [`./a2ui-renderer.service`](./a2ui-renderer.service.ts.md) — `A2uiRendererService`
- [`./component-binder.service`](./component-binder.service.ts.md) — `ComponentBinder`

## Exports

없음 (테스트 전용 파일).

## 테스트 케이스 명세

### `@Component TestTextComponent` (테스트용 인라인 컴포넌트)

- 셀렉터: `test-text`
- 템플릿: `props?.["text"]?.value()`를 `<div>`로 렌더링
- `standalone: true`, `ChangeDetectionStrategy.OnPush`
- 입력: `props: any`, `surfaceId?: string`, `componentId?: string`, `dataContextPath?: string`

---

### `describe('SurfaceComponent')`

#### `beforeEach`

- `mockRendererService` 생성: `surfaceGroup.getSurface`가 `ComponentModel('root', 'Text', {text: {value: 'Hello'}})` 하나를 포함하는 `componentsModel` Map과, `TestTextComponent`를 등록한 `catalog`를 반환하도록 설정.
- `mockBinder` 생성: `jasmine.createSpyObj`로 `bind` 스파이 포함.
- `TestBed.configureTestingModule`에서 `A2uiRendererService`와 `ComponentBinder`를 목 객체로 교체.
- `SurfaceComponent` 픽스처 생성.

#### `it('should create')`

**검증 동작**: `surfaceId` 입력을 `'test-surface'`로 설정한 뒤 `detectChanges()` 호출 시 컴포넌트 인스턴스가 truthyである.

**픽스처**: `fixture.componentRef.setInput('surfaceId', 'test-surface')`

---

#### `it('should render component-host with correct inputs')`

**검증 동작**: `surfaceId`와 `dataContextPath` 입력이 내부 `ComponentHostComponent`로 올바르게 전달되는지 확인한다.

**검증 단계**:
1. `surfaceId = 'test-surface'`, `dataContextPath = '/custom/path'` 설정 후 `detectChanges()`.
2. `By.directive(ComponentHostComponent)`로 호스트 요소 쿼리.
3. `host.componentInstance.surfaceId()`가 `'test-surface'`이어야 한다.
4. `host.componentInstance.componentKey()`가 `{ id: 'root', basePath: '/custom/path' }`이어야 한다.

---

#### `it('should use default dataContextPath of "/"')`

**검증 동작**: `dataContextPath`를 명시하지 않으면 기본값 `'/'`가 `ComponentHostComponent`에 전달된다.

**검증 단계**:
1. `surfaceId = 'test-surface'`만 설정 후 `detectChanges()`.
2. `host.componentInstance.componentKey()`가 `{ id: 'root', basePath: '/' }`이어야 한다.

## 동작 흐름

각 테스트는 `TestBed`를 통해 `SurfaceComponent`를 격리된 환경에서 생성하고, 입력 신호를 설정한 뒤 변경 감지를 트리거한다. DOM 내의 `ComponentHostComponent` 인스턴스를 `By.directive`로 조회하여 해당 컴포넌트의 Signal 입력 값을 직접 검사하는 방식으로 동작을 확인한다.
