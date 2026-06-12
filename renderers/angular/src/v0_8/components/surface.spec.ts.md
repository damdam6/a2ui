# renderers/angular/src/v0_8/components/surface.spec.ts

## 개요

`Surface` 컴포넌트의 단위 테스트 파일이다. `Renderer` 디렉티브를 `MockRenderer`로 교체하고 `MessageProcessor`를 모킹하여, 서피스 데이터 소스(외부 입력 vs. 프로세서 내부 Map)에 따른 루트 컴포넌트 렌더링 동작을 검증한다.

## 의존성

### 외부 패키지
- `@angular/core/testing`: `ComponentFixture`, `TestBed`
- `@angular/core`: `Directive`, `Input`

### 저장소 내부 모듈
- [`./surface`](./surface.ts.md) — 테스트 대상 `Surface` 컴포넌트
- [`../data/processor`](../data/processor.ts.md) — `MessageProcessor` (모킹 대상)
- [`../types`](../types.ts.md) — `AnyComponentNode`, `Surface as SurfaceType` 타입

## Exports

없음 (테스트 파일)

## 테스트 케이스

### `MockRenderer` 디렉티브
`selector: '[a2ui-renderer]'`인 standalone 디렉티브. `surfaceId: string`와 `component: any` `@Input`을 가진다. 정적 배열 `MockRenderer.instances`에 생성된 인스턴스를 추적한다.

### 픽스처 및 모킹 설정 (`beforeEach`)
- `surfacesMap`: `new Map<string, SurfaceType>()`.
- `mockProcessor`: `jasmine.createSpyObj`로 `getSurfaces`, `version` 스파이 생성. `getSurfaces`는 `surfacesMap`을 반환.
- `mockRootComponent`: `{id:'root-1', type:'Column', properties:{}}`.
- `mockSurfaceData`: `{id:'surface-1', componentTree: mockRootComponent}`.
- `TestBed` 구성: `Surface` 임포트, `MessageProcessor`(스파이) 프로바이더.
- `overrideComponent(Surface, {set: {imports: [MockRenderer]}})`: 내부 `Renderer`를 `MockRenderer`로 교체.
- `MockRenderer.instances = []` 초기화. `surfaceId='surface-1'`으로 입력 설정.
- 주의: `fixture.detectChanges()`는 `beforeEach`에서 호출하지 않고 각 테스트에서 수동으로 호출한다.

---

### `'should create'`
- **검증**: 컴포넌트 인스턴스가 truthy인지 확인.

---

### `'should render root component from surfaceInput if provided'`
- **검증 동작**: `surface` 입력을 `mockSurfaceData`로 설정하고 `detectChanges()` 후 `MockRenderer.instances.length === 1`, `instances[0].component === mockRootComponent`, `instances[0].surfaceId === 'surface-1'`인지 확인.
- **의미**: 외부 `surface` 입력이 제공되면 `getSurfaces()` Map을 조회하지 않고 해당 데이터를 우선 사용한다.

---

### `'should render root component from processor if surfaceInput is null'`
- **검증 동작**: `surfacesMap.set('surface-1', mockSurfaceData)` 설정 후 `surface=null`로 입력하고 `detectChanges()`를 실행하면 `MockRenderer.instances.length === 1`이고 `instances[0].component === mockRootComponent`인지 확인.
- **의미**: `surfaceInput`이 null이면 `processor.getSurfaces().get(surfaceId)`에서 서피스 데이터를 가져온다.

---

### `'should NOT render anything if surface or componentTree is missing'`
- **검증 동작**: `surface=null`이고 `surfacesMap`이 비어 있을 때 `detectChanges()` 후 `MockRenderer.instances.length === 0`인지 확인.
- **의미**: 서피스 데이터가 없으면 `rootComponent`가 null이 되어 렌더러가 인스턴스화되지 않는다.

## 동작 흐름

`beforeEach`에서 `detectChanges()`를 생략하여 각 테스트가 원하는 입력 상태를 설정한 뒤 직접 `detectChanges()`를 호출한다. 이를 통해 서피스 데이터 소스 우선순위(외부 입력 > 프로세서 Map)를 명확하게 검증할 수 있다.
