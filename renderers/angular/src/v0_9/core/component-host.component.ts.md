# renderers/angular/src/v0_9/core/component-host.component.ts

## 개요

A2UI 서피스 모델에 정의된 컴포넌트를 동적으로 렌더링하는 Angular 컴포넌트다. 카탈로그에서 컴포넌트 타입을 조회하고, `ComponentBinder`를 통해 reactive props 바인딩을 생성한 뒤 `NgComponentOutlet`으로 해당 컴포넌트를 인스턴스화한다. 서피스/컴포넌트 미발견 시 경고를 출력하고, 컴포넌트 모델 업데이트에 반응하여 props를 재바인딩한다.

## 의존성

### 외부 패키지
- `@angular/core`: `ChangeDetectionStrategy`, `ChangeDetectorRef`, `Component`, `DestroyRef`, `Type`, `inject`, `input`, `effect`
- `@angular/common`: `NgComponentOutlet`
- `@a2ui/web_core/v0_9`: `ComponentContext`, `ComponentModel`, `SurfaceModel`, `Subscription`

### 저장소 내부 모듈
- [`./a2ui-renderer.service`](./a2ui-renderer.service.ts.md)
- [`../catalog/types`](../catalog/types.ts.md) — `AngularCatalog`
- [`./component-binder.service`](./component-binder.service.ts.md)
- [`./types`](./types.ts.md) — `BoundProperty`

## Exports

| 이름 | 종류 |
|------|------|
| `ComponentHostComponent` | Angular 컴포넌트 클래스 |

## 상세 명세

### `ComponentHostComponent`

**데코레이터**: `@Component`  
**selector**: `'a2ui-v09-component-host'`  
**imports**: `[NgComponentOutlet]`  
**host**: `{ style: 'display: contents;' }` — 컴포넌트 자체는 레이아웃에 영향을 주지 않음  
**changeDetection**: `ChangeDetectionStrategy.OnPush`

#### 템플릿

`componentType`이 truthy일 때만 `*ngComponentOutlet`을 렌더링한다. `inputs`로 `props`, `surfaceId()`, `resolvedComponentId`, `resolvedDataContextPath`를 전달한다.

#### 입력 (input)

| 이름 | 타입 | 기본값 | 설명 |
|------|------|--------|------|
| `componentKey` | `string \| { id: string; basePath: string }` | `'root'` | 렌더링할 컴포넌트 키. 문자열이면 ID로, 객체이면 `id`와 `basePath`로 해석. |
| `surfaceId` | `string` (required) | — | 컴포넌트가 속한 서피스 ID |

#### 필드

| 이름 | 접근 | 타입 | 설명 |
|------|------|------|------|
| `rendererService` | private readonly | `A2uiRendererService` (inject) | 서피스 그룹 접근용 |
| `binder` | private readonly | `ComponentBinder` (inject) | props 바인딩용 |
| `destroyRef` | private readonly | `DestroyRef` (inject) | 소멸 시 구독 해제용 |
| `cdr` | private readonly | `ChangeDetectorRef` (inject) | 수동 변경 감지 트리거용 |
| `componentType` | protected | `Type<unknown> \| null` | 현재 렌더링할 컴포넌트 클래스. 초기값 null. |
| `props` | protected | `Record<string, BoundProperty>` | 바인딩된 프로퍼티 맵. 초기값 `{}`. |
| `context` | private | `ComponentContext \| undefined` | 현재 컴포넌트의 데이터 컨텍스트. |
| `resolvedComponentId` | protected | `string` | 실제 사용된 컴포넌트 ID. 초기값 `''`. |
| `resolvedDataContextPath` | protected | `string` | 실제 데이터 컨텍스트 경로. 초기값 `'/'`. |
| `propsSub` | private | `Subscription \| undefined` | 컴포넌트 모델 업데이트 구독. |
| `createSub` | private | `Subscription \| undefined` | 컴포넌트 생성 대기 구독. |

#### `constructor()`

1. `effect(() => { ... })`를 등록하여 `componentKey()` 또는 `surfaceId()`가 변경될 때마다 `setupComponent(key, surfaceId)`를 호출한다.
2. `destroyRef.onDestroy(() => { this.propsSub?.unsubscribe(); this.createSub?.unsubscribe(); })`를 등록하여 컴포넌트 소멸 시 모든 구독을 해제한다.

#### `private setupComponent(key, surfaceId): void`

1. `resetState()`를 호출하여 이전 상태를 초기화한다.
2. `rendererService.surfaceGroup?.getSurface(surfaceId)`로 서피스를 조회한다. 없으면 `console.warn('Surface ${surfaceId} not found')`를 출력하고 반환.
3. `key`가 객체이면 `id = key.id`, `basePath = key.basePath || '/'`, 문자열이면 `id = key`, `basePath = '/'`로 설정.
4. `resolvedComponentId = id`로 저장.
5. `surface.componentsModel.get(id)`로 컴포넌트 모델을 조회한다. 없으면 `console.warn('Component ${id} not found in surface ${surfaceId}. Waiting for it...')`를 출력하고 `surface.componentsModel.onCreated`를 구독하여 해당 ID가 생성되면 `initializeComponent()`를 호출한 뒤 구독을 해제한다. 구독은 `createSub`에 저장.
6. 컴포넌트 모델이 있으면 즉시 `initializeComponent(surface, componentModel, id, basePath)`를 호출한다.

#### `private initializeComponent(surface, componentModel, id, basePath): void`

1. `surface.catalog`를 `AngularCatalog`로 캐스팅하고 `catalog.components.get(componentModel.type)`으로 API를 조회한다. 없으면 `console.error('Component type "${componentModel.type}" not found in catalog "${catalog.id}"')`를 출력하고 반환.
2. `this.componentType = api.component`로 렌더링할 Angular 컴포넌트 클래스를 설정한다.
3. `new ComponentContext(surface, id, basePath)`로 컨텍스트를 생성하고 `this.context`에 저장.
4. `this.props = this.binder.bind(this.context)`로 props를 바인딩한다.
5. `this.resolvedDataContextPath = this.context.dataContext.path`로 경로를 저장.
6. `componentModel.onUpdated.subscribe(() => { this.props = this.binder.bind(this.context!); this.cdr.markForCheck(); })`를 `propsSub`에 저장하여 프로퍼티 추가/변경에 반응한다.
7. `this.cdr.markForCheck()`를 호출하여 변경 감지를 트리거한다.

#### `private resetState(): void`

`propsSub`와 `createSub` 구독을 모두 해제한다. `componentType = null`, `props = {}`, `resolvedDataContextPath = '/'`로 초기화하고, `cdr.markForCheck()`를 호출하여 이전 렌더링이 제거되도록 한다.

## 동작 흐름

1. Angular가 `componentKey` 또는 `surfaceId` 입력 signal을 변경할 때 `effect()`가 발동하여 `setupComponent()`를 호출한다.
2. `setupComponent()`는 서피스와 컴포넌트 모델을 순서대로 조회하고, 컴포넌트 모델이 아직 생성되지 않았으면 생성 이벤트를 대기한다.
3. `initializeComponent()`는 카탈로그에서 Angular 컴포넌트 타입을 해석하고, `ComponentBinder`로 props를 바인딩하며, `OnPush` 변경 감지 흐름에서 뷰를 갱신한다.
4. 컴포넌트 모델의 `onUpdated` 이벤트가 발생하면 props를 재바인딩하여 새로운 프로퍼티가 즉시 반영된다.
5. 컴포넌트 소멸 시 또는 `key`/`surfaceId` 변경 시 `resetState()`가 구독을 정리하고 UI를 초기화한다.
