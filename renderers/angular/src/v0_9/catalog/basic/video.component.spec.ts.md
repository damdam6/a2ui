# renderers/angular/src/v0_9/catalog/basic/video.component.spec.ts

## 개요

`VideoComponent`의 단위 테스트 파일이다. Angular `TestBed`를 사용하여 `A2uiRendererService`와 `ComponentBinder`를 목(mock)으로 제공하고, 비디오 렌더링 및 누락된 props 처리 동작을 검증한다.

## 의존성

### 외부 패키지
- `@angular/core/testing`: `ComponentFixture`, `TestBed`

### 저장소 내부 모듈
- [`./video.component`](./video.component.ts.md)
- [`../../core/a2ui-renderer.service`](../../core/a2ui-renderer.service.ts.md)
- [`../../core/component-binder.service`](../../core/component-binder.service.ts.md)
- [`../../core/test-utils`](../../core/test-utils.ts.md) — `setComponentProps`, `createBoundProperty`, `ComponentToProps`

## Exports

없음 (테스트 파일).

## 테스트 케이스 명세

### 픽스처 및 모킹 설정

**`beforeEach` (async)**: `TestBed`를 구성한다.
- `A2uiRendererService`를 목으로 대체: `surfaceGroup.getSurface`가 `componentsModel: new Map()`과 `catalog: { id: 'mock-catalog', components: new Map() }`를 가진 객체를 반환하는 `jasmine.createSpy`로 설정.
- `ComponentBinder`를 `bind` 메서드를 가진 `jasmine.createSpyObj`로 대체.
- `VideoComponent`를 `imports`에 추가하고 두 모의 객체를 `providers`에 등록.

**`beforeEach` (sync)**: 각 테스트 전에 컴포넌트 픽스처를 생성하고,
- `surfaceId` 입력에 `'test-surface'`를 설정,
- `dataContextPath` 입력에 `'/'`를 설정,
- `defaultProps`를 `{ url: createBoundProperty('https://example.com/video.mp4') }`로 구성하여 `setComponentProps`로 주입한다.

---

### `should create`

**검증 동작**: 컴포넌트 인스턴스가 truthy임을 확인한다.  
**픽스처/모킹**: 기본 설정 사용. `fixture.detectChanges()` 호출 후 `expect(component).toBeTruthy()`.

---

### `should render video with url`

**검증 동작**: DOM에 `<video>` 엘리먼트가 렌더링되고 `src` 속성이 truthy임을 확인한다.  
**픽스처/모킹**: 기본 `defaultProps`(`url: 'https://example.com/video.mp4'`) 사용. `fixture.nativeElement.querySelector('video')`로 `HTMLVideoElement`를 조회하여 `src`가 truthy인지 검사.

---

### `should handle missing props`

**검증 동작**: props를 빈 객체로 설정했을 때 `<video>` 엘리먼트의 `src` 어트리뷰트가 falsy(null)임을 확인한다.  
**픽스처/모킹**: `setComponentProps(fixture, {} as ComponentToProps<VideoComponent>)`로 props를 빈 상태로 재설정. `video.getAttribute('src')`가 falsy임을 검사.

## 동작 흐름

테스트는 `VideoComponent`를 독립적으로 마운트하고, 서비스 의존성을 목으로 차단한다. 각 케이스는 `fixture.detectChanges()`를 호출하여 Angular 변경 감지를 트리거한 뒤 DOM 상태 또는 컴포넌트 인스턴스를 assertion한다.
