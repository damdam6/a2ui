# renderers/react/src/v0_9/adapter.tsx

## 개요

A2UI v0_9 React 어댑터 레이어를 정의하는 파일이다. `@a2ui/web_core/v0_9`의 제네릭 바인딩 인프라(`GenericBinder`)를 React의 상태 구독 메커니즘(`useSyncExternalStore`)과 연결하는 두 팩토리 함수를 제공한다. 컴포넌트 구현자는 이 팩토리 함수를 사용하여 A2UI 데이터 모델에 반응형으로 연결된 React 컴포넌트를 생성한다.

## 의존성

### 외부 패키지

- `react` — `React`, `useRef`, `useSyncExternalStore`, `useCallback`, `memo`, `useEffect`

### 저장소 내부 모듈

- `@a2ui/web_core/v0_9` — `ComponentContext`, `GenericBinder`, `ComponentApi`, `InferredComponentApiSchemaType`, `ResolveA2uiProps` 타입

## Exports

| 이름 | 종류 | 설명 |
|------|------|------|
| `ReactComponentImplementation` | 인터페이스 | `ComponentApi`를 확장한 React 전용 컴포넌트 구현체 타입 |
| `ReactA2uiComponentProps<T>` | 타입 별칭 | React 컴포넌트가 받는 props 타입 (resolved props + buildChild + context) |
| `createComponentImplementation` | 함수 | `GenericBinder` 기반의 React 컴포넌트 구현체 팩토리 |
| `createBinderlessComponentImplementation` | 함수 | 바인더 없이 context를 직접 관리하는 컴포넌트 구현체 팩토리 |

## 상세 명세

### 인터페이스 `ReactComponentImplementation extends ComponentApi`

`ComponentApi`(`name`, `schema` 필드 포함)를 확장하며 React 전용 필드를 추가한다.

| 필드 | 타입 | 설명 |
|------|------|------|
| `render` | `React.FC<{ context: ComponentContext; buildChild: (id: string, basePath?: string) => React.ReactNode }>` | 프레임워크 전용 렌더링 래퍼 컴포넌트 |

### 타입 별칭 `ReactA2uiComponentProps<T>`

```
{
  props: T;
  buildChild: (id: string, basePath?: string) => React.ReactNode;
  context: ComponentContext;
}
```

`T`는 API 스키마에서 추론된 resolved props 타입이다.

### `createComponentImplementation<Api extends ComponentApi>(api, RenderComponent): ReactComponentImplementation`

**제네릭:** `Api extends ComponentApi`  
**매개변수:**
- `api: Api` — 컴포넌트 API 정의 (`name`, `schema` 포함)
- `RenderComponent: React.FC<ReactA2uiComponentProps<ResolveA2uiProps<InferredComponentApiSchemaType<Api>>>>` — 실제 UI를 렌더링하는 React 컴포넌트

**반환 타입:** `ReactComponentImplementation`

**내부 타입:** `type Props = ResolveA2uiProps<InferredComponentApiSchemaType<Api>>` — API 스키마에서 추론된 props 타입.

**내부 컴포넌트 `MemoizedRender`:**  
`React.memo(RenderComponent, comparator)`로 생성된다. 커스텀 비교 함수는 다음 조건 중 하나라도 해당하면 `false`(리렌더 필요)를 반환한다:
- `prev.props !== next.props`
- `prev.context.componentModel.id !== next.context.componentModel.id`
- `prev.context.dataContext.path !== next.context.dataContext.path`

세 조건 모두 동일하면 `true`(리렌더 불필요)를 반환한다.

**내부 컴포넌트 `ReactWrapper`:**  
`React.FC<{ context: ComponentContext; buildChild: ... }>`로 정의된다.

동작 단계:
1. `useRef<GenericBinder<Props> | null>(null)`로 바인더 참조를 유지한다.
2. 렌더 시점에 바인더 동기화:
   - `bindingRef.current`가 `null`이면 `new GenericBinder<Props>(context, api.schema)`를 생성한다.
   - `bindingRef.current`가 있지만 보유한 `context` 참조가 현재 `context`와 다르면 기존 바인더를 `dispose()`하고 새 바인더를 생성한다. (`(bindingRef.current as unknown as {context: ComponentContext}).context !== context` 패턴 사용)
3. `useCallback`으로 `subscribe` 함수를 생성한다: `binding.subscribe(callback)` 후 해제 함수를 반환한다. 의존성: `[binding]`.
4. `useCallback`으로 `getSnapshot` 함수를 생성한다: `() => binding.snapshot`. 의존성: `[binding]`.
5. `useSyncExternalStore(subscribe, getSnapshot)`으로 `props`를 구독한다.
6. `useEffect`로 언마운트 시 `binding.dispose()`를 호출하여 DataModel 구독 누수를 방지한다. 의존성: `[binding]`.
7. `<MemoizedRender props={props || ({} as Props)} buildChild={buildChild} context={context} />`를 렌더링한다. `props`가 nullish이면 빈 객체를 폴백으로 사용한다.

최종 반환 객체: `{ name: api.name, schema: api.schema, render: ReactWrapper }`.

### `createBinderlessComponentImplementation(api, RenderComponent): ReactComponentImplementation`

**매개변수:**
- `api: ComponentApi` — 컴포넌트 API 정의
- `RenderComponent: React.FC<{ context: ComponentContext; buildChild: (id: string, basePath?: string) => React.ReactNode }>` — context를 직접 관리하는 React 컴포넌트

**반환 타입:** `ReactComponentImplementation`

`GenericBinder` 없이 `RenderComponent`를 `render`로 직접 사용한다. 컴포넌트 자체적으로 context 바인딩을 관리해야 할 때 사용한다. 반환 객체: `{ name: api.name, schema: api.schema, render: RenderComponent }`.

## 동작 흐름

1. 컴포넌트 구현자는 `createComponentImplementation(api, RenderComponent)`를 호출하여 `ReactComponentImplementation`을 생성한다.
2. 생성된 구현체는 `surface.catalog.components`에 등록된다.
3. `A2uiSurface.tsx`의 `DeferredChild`가 카탈로그에서 구현체를 조회하여 `compImpl.render`(= `ReactWrapper`)를 호출한다.
4. `ReactWrapper`는 `GenericBinder`를 통해 A2UI 데이터 모델과 동기화된 `props`를 유지하며, 데이터 변경 시 `useSyncExternalStore`가 자동으로 리렌더를 트리거한다.
5. `RenderComponent`는 최종적으로 type-safe한 `props`, `buildChild`, `context`를 받아 UI를 렌더링한다.
