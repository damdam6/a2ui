# renderers/react/tests/v0_9/adapter.test.tsx

## 개요

v0.9 React 렌더러 어댑터(`createComponentImplementation`)와 `A2uiSurface`의 핵심 동작을 검증하는 통합/단위 테스트 파일이다. props 해석, 데이터 모델 반응성, 언마운트 시 리스너 정리, 그리고 progressive streaming 환경에서의 점진적 렌더링(stale closure 방지)을 중점적으로 테스트한다.

## 의존성

### 외부 패키지
- `vitest`: `describe`, `it`, `expect`, `vi`
- `@testing-library/react`: `render`, `screen`, `act`
- `@a2ui/web_core/v0_9`: `ComponentContext`, `ComponentModel`, `SurfaceModel`, `Catalog`, `CommonSchemas`
- `zod`: `z`

### 저장소 내부 모듈
- `../../src/v0_9/adapter` — `createComponentImplementation`
- `../../src/v0_9/A2uiSurface` — `A2uiSurface`

## Exports

없음 (테스트 파일).

## 테스트 케이스 명세

### 픽스처

- `mockCatalog`: `new Catalog('test', [], [])` — 빈 컴포넌트 목록의 카탈로그 인스턴스. 모든 테스트에서 기본 surface 생성에 사용.

### `describe('adapter')`

---

#### `it('should render component with resolved props')`

**검증 동작**: `createComponentImplementation`으로 만든 컴포넌트가 `ComponentContext`에서 props를 해석하고 자식 컴포넌트도 `buildChild`로 렌더링하는지 확인.

**과정**:
1. `SurfaceModel`과 `ComponentModel('c1', 'TestComp', {text: 'Hello World', child: 'child1'})` 생성 후 surface에 추가.
2. `ComponentContext(surface, 'c1', '/')` 생성.
3. `TestApiDef`: schema에 `text: CommonSchemas.DynamicString`, `child: CommonSchemas.ComponentId` 정의.
4. `createComponentImplementation`으로 `TestComponent` 생성. 렌더 함수는 `<div><span>{props.text}</span>{props.child && buildChild(props.child)}</div>` 반환.
5. `buildChild`를 `vi.fn().mockImplementation(id => <div data-testid={id}>Child</div>)`로 목킹.
6. `<TestComponent.render context={context} buildChild={buildChild} />` 렌더링.
7. `screen.getByText('Hello World')` 및 `screen.getByTestId('child1')` 존재 검증.

---

#### `it('should react to data model changes')`

**검증 동작**: 컴포넌트가 `{path: '/greeting'}` 동적 값을 사용할 때 데이터 모델 업데이트에 자동 반응하여 재렌더링되는지 확인.

**과정**:
1. `ComponentModel('c1', 'TestComp', {text: {path: '/greeting'}})` 생성 후 surface에 추가.
2. `surface.dataModel.set('/greeting', 'Hello Reactive')`로 초기 데이터 설정.
3. `ComponentContext` 생성 후 컴포넌트를 렌더링 → `data-testid="msg"` 엘리먼트의 textContent가 `'Hello Reactive'`임을 확인.
4. `act(async () => { surface.dataModel.set('/greeting', 'Updated Greeting'); })`로 데이터 갱신.
5. 동일 엘리먼트 textContent가 `'Updated Greeting'`으로 변경되었는지 확인.

---

#### `it('should clean up listeners on unmount')`

**검증 동작**: 컴포넌트 언마운트 시 `subscribeDynamicValue`가 반환한 `unsubscribe` 함수가 호출되는지 확인 (메모리 누수 방지).

**과정**:
1. `ComponentModel('c1', 'TestComp', {text: {path: '/greeting'}})` 생성.
2. `unsubscribeSpy = vi.fn()`.
3. `vi.spyOn(context.dataContext, 'subscribeDynamicValue').mockReturnValue({value: 'initial', unsubscribe: unsubscribeSpy})`로 구독 함수를 스파이로 교체.
4. 컴포넌트 렌더링 → `spyAddListener`가 호출되었는지 확인.
5. `unmount()` 후 `unsubscribeSpy`가 호출되었는지 확인.

---

#### `it('preserves progressive rendering (avoids stale closures from over-memoization)')`

**검증 동작**: 스트리밍 환경에서 자식 컴포넌트가 나중에 추가될 때, 부모 컴포넌트를 재렌더링하지 않고 `DeferredChild` 래퍼가 자체적으로 해석하는지 확인.

**과정**:
1. `ParentApiDef`(schema: `{child: CommonSchemas.ComponentId}`)와 `ChildApiDef`(schema: `{text: CommonSchemas.DynamicString}`)를 정의.
2. `parentRenderCount` 카운터를 선언. `TestParent` 렌더 함수에서 매 렌더마다 증가.
3. `testCatalog = new Catalog('test', [TestParent, TestChild], [])` 생성.
4. `new SurfaceModel('test-surface', testCatalog)` 생성.
5. **초기 상태**: `root` 컴포넌트(`TestParent`, `child: 'child1'`)만 추가. 자식 `child1`은 아직 없음.
6. `<A2uiSurface surface={surface} />` 렌더링.
7. `getByTestId('parent').textContent`에 `'[Loading child1...]'` 포함됨을 확인 (fallback 동작).
8. `countBeforeChild = parentRenderCount` 저장.
9. `act(async () => { surface.componentsModel.addComponent(new ComponentModel('child1', 'TestChild', {text: 'Loaded Data'})); })`로 자식 추가.
10. `queryByTestId('resolved')` 존재 및 textContent가 `'Loaded Data'`임 확인.
11. `parentRenderCount === countBeforeChild` — 부모가 재렌더링되지 않았음을 확인.

## 동작 흐름

이 파일은 어댑터 레이어의 세 가지 핵심 품질 속성을 순서대로 검증한다: (1) props 정확한 해석, (2) 반응형 데이터 바인딩, (3) 자원 정리. 마지막 케이스는 React 렌더링 최적화(stale closure 방지, DeferredChild 구조)의 정확성을 회귀 테스트로 보호한다.
