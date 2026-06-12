# renderers/lit/src/v0_9/surface/a2ui-surface.ts

## 개요

`a2ui-surface` 커스텀 엘리먼트를 정의하는 파일로, A2UI의 최상위 렌더링 컨테이너 역할을 한다. `SurfaceModel`을 프로퍼티로 받아 루트 컴포넌트가 준비되면 `renderA2uiNode`를 통해 컴포넌트 트리 전체를 렌더링한다. 루트 컴포넌트가 아직 생성되지 않은 상태에서는 로딩 상태를 표시하고, `onCreated` 이벤트를 구독하여 루트가 나타나면 자동으로 렌더링을 갱신한다.

## 의존성

### 외부 패키지
- `lit`: `html`, `nothing`, `LitElement`, `PropertyValues`
- `lit/decorators.js`: `customElement`, `property`, `state`
- `@a2ui/web_core/v0_9`: `SurfaceModel`, `ComponentContext`
- `@a2ui/lit/v0_9`: `LitComponentApi`

### 저장소 내부 모듈
- [`./render-a2ui-node.js`](./render-a2ui-node.ts.md) — `renderA2uiNode` 함수

## Exports

- `A2uiSurface` (클래스): `a2ui-surface` 커스텀 엘리먼트

## 상세 명세

### `A2uiSurface` 클래스

`LitElement`를 직접 상속하는 Lit 커스텀 엘리먼트. `@customElement('a2ui-surface')` 데코레이터로 등록된다.

#### 프로퍼티 및 상태 필드

- `@property({type: Object}) accessor surface: SurfaceModel<LitComponentApi> | undefined`
  - 외부에서 주입하는 서피스 모델. 컴포넌트 트리와 카탈로그를 포함한다.
- `@state() accessor _hasRoot: boolean = false`
  - 내부 상태. `surface.componentsModel`에 `'root'` ID를 가진 컴포넌트가 존재하는지 여부.
- `private unsubscribe?: () => void`
  - `onCreated` 구독 해제 함수 참조. `undefined`이면 활성 구독 없음.

#### `protected willUpdate(changedProperties: PropertyValues): void`
- 시그니처: `willUpdate(changedProperties: PropertyValues): void`
- `changedProperties`에 `'surface'`가 포함된 경우에만 동작한다.
- 기존 `this.unsubscribe`가 있으면 호출하여 이전 구독을 해제하고 `undefined`로 초기화한다.
- `this.surface?.componentsModel.get('root')`의 truthy 여부로 `_hasRoot`를 즉시 설정한다.
- `surface`가 존재하고 `_hasRoot`가 `false`이면 `surface.componentsModel.onCreated.subscribe(comp => {...})`로 새 구독을 시작한다:
  - 콜백 내에서 `comp.id === 'root'`인 경우 `_hasRoot = true`로 설정하고 `this.requestUpdate()`를 호출한다.
  - 이후 `this.unsubscribe?.()`로 스스로 구독을 해제하고 `undefined`로 초기화한다.
- 구독 반환값의 `unsubscribe` 메서드를 `this.unsubscribe`에 저장한다.

#### `disconnectedCallback(): void`
- 시그니처: `disconnectedCallback(): void`
- `super.disconnectedCallback()`을 먼저 호출한다.
- `this.unsubscribe`가 있으면 호출하여 활성 구독을 해제하고 `undefined`로 초기화한다. 메모리 누수 방지를 위한 정리 로직.

#### `render(): TemplateResult | typeof nothing`
- 시그니처: `render(): TemplateResult | typeof nothing`
- `this.surface`가 없으면 `nothing`을 반환한다.
- `this._hasRoot`가 `false`이면 로딩 슬롯 `html`<slot name="loading"><div>Loading surface...</div></slot>``를 반환한다.
- 그 외 경우: `try-catch` 블록 안에서 `new ComponentContext(this.surface, 'root', '/')`를 생성하고 `renderA2uiNode(rootContext, this.surface.catalog)`를 호출하여 결과를 반환한다.
- 오류가 발생하면 `console.error`로 기록하고 `html`<div>Error rendering surface</div>``를 반환한다.

## 동작 흐름

1. 부모 컴포넌트가 `surface` 프로퍼티를 설정하면 `willUpdate()`가 호출된다.
2. `root` 컴포넌트가 이미 있으면 즉시 렌더링, 없으면 `onCreated` 이벤트 구독으로 대기한다.
3. `root` 컴포넌트가 추가되면 구독 콜백이 실행되어 `_hasRoot = true`로 상태를 갱신하고 `requestUpdate()`로 재렌더링을 트리거한다.
4. `render()`에서 `renderA2uiNode`가 루트부터 하위 컴포넌트 트리를 재귀적으로 렌더링한다.
5. 엘리먼트가 DOM에서 제거될 때 `disconnectedCallback()`이 구독을 정리한다.
