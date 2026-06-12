# renderers/angular/src/v0_8/rendering/renderer.ts

## 개요

v0.8 Angular 렌더러의 핵심 지시자(`Renderer`)를 구현한다. 컴포넌트 노드 데이터를 받아 카탈로그에서 해당 Angular 컴포넌트를 찾아 `ViewContainerRef`에 동적으로 생성하며, 동일 ID/타입의 컴포넌트는 DOM을 파괴하지 않고 입력만 갱신하는 focus 보존 최적화를 적용한다. 최초 생성 시 `structuralStyles`를 `document.head`에 한 번만 주입한다.

## 의존성

### 외부 패키지

- `@angular/core` — `Directive`, `effect`, `inject`, `input`, `ViewContainerRef`, `Type`, `PLATFORM_ID`, `ComponentRef`
- `@angular/common` — `DOCUMENT`, `isPlatformBrowser`
- `@a2ui/web_core/styles/index` — `structuralStyles`

### 저장소 내부 모듈

- [`./catalog`](catalog.ts.md) — `Catalog`
- [`../data`](../data/index.ts.md) — `MessageProcessor`
- [`../types`](../types.ts.md) — `AnyComponentNode`, `SurfaceID` (타입 전용)
- [`./dynamic-component`](dynamic-component.ts.md) — `DynamicComponent`

## Exports

| 이름 | 종류 |
|---|---|
| `Renderer` | 클래스 (Angular `@Directive`) |

## 상세 명세

### `Renderer` 클래스

데코레이터: `@Directive({ selector: '[a2ui-renderer]', standalone: true })`

#### 정적 필드

- `private static hasInsertedStyles: boolean = false` — 프로세스 전체에서 스타일 태그를 중복 삽입하지 않도록 막는 플래그.

#### 주입된 의존성

- `catalog: Catalog` — `inject(Catalog)`, private readonly. 컴포넌트 타입 이름 → 컴포넌트 클래스 매핑.
- `container: ViewContainerRef` — `inject(ViewContainerRef)`, private readonly. 동적 컴포넌트를 생성·제거하는 뷰 컨테이너.
- `processor: MessageProcessor` — `inject(MessageProcessor)`, private readonly. 데이터 모델 버전 신호를 구독하여 effect 재실행 트리거.

#### 입력 신호

- `surfaceId = input.required<SurfaceID>()` — 서피스 ID.
- `component = input.required<AnyComponentNode>()` — 렌더링할 컴포넌트 노드.

#### 인스턴스 상태 필드

- `currentId: string | null = null` — 현재 렌더링 중인 노드의 ID.
- `currentType: string | null = null` — 현재 렌더링 중인 노드의 타입 이름.
- `currentComponentRef: ComponentRef<DynamicComponent<AnyComponentNode>> | null = null` — 현재 살아있는 컴포넌트 참조.

#### `constructor()`

동작 단계:
1. `inject(PLATFORM_ID)`, `inject(DOCUMENT)`로 플랫폼 및 문서 객체를 얻는다.
2. `!Renderer.hasInsertedStyles && isPlatformBrowser(platformId)` 조건이 참이면:
   - `document.createElement('style')` 호출.
   - `styleElement.textContent = structuralStyles` 설정.
   - `document.head.appendChild(styleElement)` 삽입.
   - `Renderer.hasInsertedStyles = true` 설정하여 이후 중복 삽입 방지.
3. `effect(...)` 등록:
   - `this.processor.version()` 읽기(데이터 모델 변경 시 effect 재실행 보장).
   - `let node = this.component()` 및 `const surfaceId = this.surfaceId()` 읽기.
   - **v0.8 래핑 형식 처리**: `!node.type && (node as any).component`이면 `Object.keys(wrapped)[0]`으로 타입을 추출하고 `{ ...node, type, properties: wrapped[type] }` 형태로 node를 재구성.
   - `id = node.id`, `type = node.type` 추출.
   - **Focus 보존 최적화**: `this.currentComponentRef && this.currentId === id && this.currentType === type`이면 `updateInputs`만 호출하고 early return.
   - 그 외: `container.clear()`, 상태 필드 초기화(`currentComponentRef = null`, `currentId = id`, `currentType = type`).
   - `this.catalog[node.type]`이 없으면 `console.error('Unknown component type: ${node.type}')` 출력 후 return.
   - `this.render(container, node, surfaceId, config)` 호출.

#### `private render(container, node, surfaceId, config)`

시그니처: `private render(container: ViewContainerRef, node: AnyComponentNode, surfaceId: string, config: any)`

동작 단계:
1. `this.resolveComponentType(config)` 호출 → `componentTypeOrPromise` 획득.
2. `componentTypeOrPromise instanceof Promise`이면:
   - `.then(componentType => { ... })` 콜백 등록.
   - 콜백 내부: `this.currentId === node.id && this.currentType === node.type` (아직 동일 노드를 렌더링해야 함) 확인 후 `container.createComponent(componentType)` → `currentComponentRef` 저장 → `updateInputs` 호출.
3. 그렇지 않고 `componentTypeOrPromise`가 truthy이면 (동기 타입): 즉시 `container.createComponent` → `currentComponentRef` 저장 → `updateInputs` 호출.

#### `private resolveComponentType(config: any): Type<unknown> | Promise<Type<unknown>> | null`

동작 단계:
1. `typeof config === 'function'`이면 `config()` 호출 후 반환.
2. `typeof config === 'object' && config !== null`이면:
   - `typeof config.type === 'function'`이면 `config.type()` 호출 후 반환.
   - 그 외 `config.type` 직접 반환.
3. 어느 분기도 해당하지 않으면 `null` 반환.

#### `private updateInputs(componentRef, node, surfaceId)`

시그니처: `private updateInputs(componentRef: ComponentRef<DynamicComponent<AnyComponentNode>>, node: AnyComponentNode, surfaceId: string)`

동작 단계:
1. `componentRef.setInput('surfaceId', surfaceId)` — 서피스 ID 주입.
2. `componentRef.setInput('component', node)` — 노드 데이터 주입.
3. `componentRef.setInput('weight', node.weight ?? 0)` — 레이아웃 가중치 주입 (없으면 `0` 기본값).
4. `node.properties`가 존재하면 각 `[key, value]` 쌍에 대해 `componentRef.setInput(key, value)` 시도:
   - 실패(`throw`)하면 `console.warn('[Renderer] Property "${key}" could not be set on component ${node.type}. ...')` 경고 출력.
5. `componentRef.changeDetectorRef.markForCheck()` 호출하여 변경 감지 예약.

## 동작 흐름

1. `Renderer` 지시자가 `[a2ui-renderer]` 속성을 가진 요소에 바인딩된다.
2. 생성자에서 구조적 스타일이 한 번만 head에 주입된다.
3. `effect`가 `component()`, `surfaceId()`, `processor.version()` 신호에 반응성을 갖고 등록된다.
4. 신호 변경 시 effect 재실행: 동일 컴포넌트 ID/타입이면 입력만 갱신(DOM 유지), 달라지면 컨테이너를 비우고 새 컴포넌트를 동기 또는 비동기로 생성한다.
5. 생성된 컴포넌트는 `updateInputs`를 통해 `surfaceId`, `component`, `weight`, 그리고 `node.properties`의 모든 속성을 `@Input`으로 받는다.
