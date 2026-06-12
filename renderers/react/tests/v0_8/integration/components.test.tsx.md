# renderers/react/tests/v0_8/integration/components.test.tsx

## 개요

컴포넌트 업데이트, 중첩 컴포넌트 구조, 그리고 에러 처리 시나리오를 검증하는 통합 테스트 파일이다. `A2UIProvider`·`A2UIRenderer`·`useA2UI` 훅을 실제로 마운트하고 `processMessages`를 호출하여 서버 메시지 처리 후 DOM 상태가 올바른지 확인한다. 비동기 시나리오는 `waitFor`로 검증하며, 순수 초기 렌더 시나리오는 동기적으로 검증한다.

## 의존성

### 외부 패키지

| 패키지 | 가져오는 항목 |
|---|---|
| `vitest` | `describe`, `it`, `expect`, `vi` |
| `@testing-library/react` | `render`, `screen`, `waitFor` |
| `react` | `React` (default), `useEffect` |
| `@a2ui/web_core/types/types` | `* as Types` (타입 전용) |

### 저장소 내부 모듈

| 경로 | 가져오는 항목 |
|---|---|
| [`../../../src/v0_8`](../../../src/v0_8.md) | `A2UIProvider`, `A2UIRenderer`, `useA2UI` |
| [`../utils`](../utils/index.ts.md) | `TestWrapper`, `TestRenderer`, `createSurfaceUpdate`, `createBeginRendering` |

> `../utils`는 `tests/v0_8/utils/index.ts`를 통해 재내보내기되는 편의 진입점이다.

## Exports

이 파일은 아무것도 내보내지 않는다 (테스트 전용 파일).

## 상세 명세 — 테스트 케이스

### `describe('Component Updates', ...)`

컴포넌트 데이터 변경, 타입 전환, 목록 조작 등 서피스 업데이트 동작을 검증하는 그룹이다.

---

#### `it('should update already-rendered surface with surfaceUpdate alone (no new beginRendering)')`

**픽스처 / 내부 컴포넌트: `UpdateWithoutBeginRenderingRenderer`**

- `useA2UI()`에서 `processMessages`를 얻고, `React.useState<'initial' | 'updated'>('initial')`로 단계를 추적한다.
- `stage === 'initial'`일 때: `id: 'text-1'` Text 컴포넌트(`'Original text'`)를 포함한 `createSurfaceUpdate` + `createBeginRendering('text-1')`을 처리한다. 10ms 후 `setStage('updated')`를 호출한다.
- `stage === 'updated'`일 때: 동일한 `id: 'text-1'`에 `'Updated text'`로 변경된 `createSurfaceUpdate`만 처리하고, **`createBeginRendering`은 전송하지 않는다**.
- 렌더링 결과: `<A2UIRenderer surfaceId="@default" />`와 단계 확인용 `<span data-testid="stage">`.

**검증 항목**

1. 초기 렌더 직후 `'Original text'`가 DOM에 존재한다.
2. `waitFor` 내에서 `stage`가 `'updated'`로 바뀐 뒤 `'Updated text'`가 존재하고 `'Original text'`는 사라진다.

**목적**: `beginRendering` 없이 `surfaceUpdate`만으로도 이미 렌더된 서피스가 갱신됨을 보장한다.

---

#### `it('should update component props when new message received')`

**픽스처 / 내부 컴포넌트: `UpdateRenderer`**

- `useEffect`에서 즉시 `'Before'` Text를 담은 `surfaceUpdate + beginRendering`을 처리한다.
- 10ms 후 동일한 `id: 'text-1'`에 `'After'`로 바뀐 `surfaceUpdate + beginRendering`을 다시 처리한다.

**검증 항목**

1. 초기 렌더 직후 `'Before'`가 존재한다.
2. `waitFor`로 `'After'`가 렌더됨을 확인한다 (테스트가 Promise를 반환).

---

#### `it('should handle component type change')`

**픽스처 / 내부 컴포넌트: `TypeChangeRenderer`**

- 초기: `id: 'comp-1'`를 Text(`'I am text'`)로 렌더한다.
- 10ms 후: `id: 'btn-text'`(Text `'Click me'`)를 추가하고, `id: 'comp-1'`을 Button(`child: 'btn-text'`, `action: {name: 'test'}`)으로 교체하는 `surfaceUpdate + beginRendering`을 전송한다.

**검증 항목**

1. 초기에 `'I am text'`가 존재한다.
2. `waitFor`로 `role='button'` + `name='Click me'`인 버튼이 렌더됨을 확인한다.

---

#### `it('should add new components to existing surface')`

**픽스처 / 내부 컴포넌트: `AddComponentRenderer`**

- 초기: `id: 'text-1'`(`'First'`) + `beginRendering('text-1')`.
- 10ms 후: `text-1`(`'First'`), `text-2`(`'Second'`), `col-1`(Column, `children: {explicitList: ['text-1', 'text-2']}`)를 담은 `surfaceUpdate + beginRendering('col-1')`.

**검증 항목**

- `waitFor`로 `'First'`와 `'Second'` 모두 DOM에 존재함을 확인한다.

---

#### `it('should remove elements from a list via surfaceUpdate')`

**픽스처 / 내부 컴포넌트: `RemoveElementsRenderer`**

- `React.useState<'initial' | 'removed'>('initial')`으로 단계 추적.
- `stage === 'initial'`: `item-1`, `item-2`, `item-3` 세 Text와 `list-1`(List, 세 항목 모두 참조)을 `surfaceUpdate + beginRendering`으로 처리한다. 10ms 후 `setStage('removed')`.
- `stage === 'removed'`: `item-2`를 제거하고 `list-1`의 `children`을 `['item-1', 'item-3']`으로 줄인 `surfaceUpdate`만 전송한다 (`beginRendering` 없음).

**검증 항목**

1. 초기: `'Item 1'`, `'Item 2'`, `'Item 3'` 모두 존재.
2. `waitFor` 내에서 `stage === 'removed'` 후: `'Item 1'`·`'Item 3'` 존재, `'Item 2'`는 DOM에서 사라짐.

---

#### `it('should reorder elements via surfaceUpdate')`

**픽스처 / 내부 컴포넌트: `ReorderElementsRenderer`**

- `React.useState<'initial' | 'reordered'>('initial')`으로 단계 추적.
- `stage === 'initial'`: `item-a`(`'A'`), `item-b`(`'B'`), `item-c`(`'C'`)와 `col-1`(Column, 순서 `['item-a', 'item-b', 'item-c']`)을 렌더한다. 10ms 후 `setStage('reordered')`.
- `stage === 'reordered'`: 동일한 세 항목과 `col-1`의 순서를 `['item-c', 'item-a', 'item-b']`로 변경한 `surfaceUpdate`만 전송 (`beginRendering` 없음).

**검증 항목**

- `waitFor`로 `stage === 'reordered'` 확인 후, `container.querySelectorAll('.a2ui-text')`로 얻은 3개 요소가 순서대로 `'C'`, `'A'`, `'B'` 텍스트를 가짐을 확인한다.

---

#### `it('should NOT empty the surface when invalid surfaceUpdate is rejected (requires deleteSurface)')`

**픽스처 / 내부 컴포넌트: `EmptySurfaceRenderer`**

- `React.useState<'initial' | 'attempted'>('initial')`으로 단계 추적.
- `stage === 'initial'`: `text-1`(`'Persistent content'`) + `col-1`(Column)을 렌더 후 10ms 뒤 `setStage('attempted')`.
- `stage === 'attempted'`: `components: []`인 빈 배열을 가진 `surfaceUpdate` 메시지를 `processMessages`에 전달한다. Zod 스키마 검증이 이를 거부하므로 `try/catch`로 예외를 무시한다.

**검증 항목**

1. 초기에 `'Persistent content'`가 존재한다.
2. `waitFor` 내에서 `stage === 'attempted'` 후에도 `'Persistent content'`가 여전히 존재하고, `data-testid="empty-fallback"`은 나타나지 않는다.

**목적**: 잘못된 메시지(스키마 검증 실패)가 기존 서피스를 파괴하지 않음을 확인. 실제 서피스 비우기는 `deleteSurface` 메시지로만 가능함을 문서화.

---

### `describe('Nested Components', ...)`

중첩 컴포넌트 구조를 정적으로 (동기 렌더로) 검증하는 그룹이다. `TestWrapper` + `TestRenderer`를 사용한다.

---

#### `it('should render deeply nested component structures')`

**픽스처**: `messages` 배열에 5개 컴포넌트를 선언한다.
- `inner-text`: Text(`'Deep content'`)
- `inner-card`: Card(`child: 'inner-text'`)
- `inner-col`: Column(`children: {explicitList: ['inner-card']}`)
- `outer-card`: Card(`child: 'inner-col'`)
- `outer-col`: Column(`children: {explicitList: ['outer-card']}`)
- `createBeginRendering('outer-col')`

**검증 항목**

1. `'Deep content'` 텍스트가 DOM에 존재한다.
2. `container.querySelectorAll('.a2ui-card')`가 정확히 2개 반환된다 (`inner-card`, `outer-card`).

---

#### `it('should handle List with multiple items')`

**픽스처**: `item-1`(`'Item 1'`), `item-2`(`'Item 2'`), `item-3`(`'Item 3'`) 세 Text와 `list-1`(List, `children: {explicitList: ['item-1', 'item-2', 'item-3']}`)을 포함하는 `messages`. `createBeginRendering('list-1')`.

**검증 항목**: `'Item 1'`, `'Item 2'`, `'Item 3'` 모두 DOM에 존재.

---

#### `it('should handle Row with mixed children')`

**픽스처**:
- `text-1`: Text(`'Label'`)
- `btn-text`: Text(`'Action'`)
- `btn-1`: Button(`child: 'btn-text'`, `action: {name: 'act'}`)
- `icon-1`: Icon(`name: {literalString: 'home'}`)
- `row-1`: Row(`children: {explicitList: ['text-1', 'btn-1', 'icon-1']}`)
- `createBeginRendering('row-1')`

**검증 항목**

1. `'Label'` 텍스트 존재.
2. `role='button'` + `name='Action'` 버튼 존재.
3. `container.querySelector('.a2ui-icon')`이 null이 아님.

---

### `describe('Error Handling', ...)`

컴포넌트 데이터 오류 및 정상 렌더 시 에러 없음을 검증하는 그룹이다.

---

#### `it('should throw error for invalid component data')`

**모킹**: `vi.spyOn(console, 'error').mockImplementation(() => {})` — React 에러 경계 콘솔 출력 억제.

**픽스처**: `btn-1`(Button, `child: 'non-existent'` — 존재하지 않는 자식 ID 참조) + `createBeginRendering('btn-1')`.

**검증 항목**: `render(...)` 호출이 예외를 던짐(`expect(...).toThrow()`). 테스트 후 `consoleSpy.mockRestore()`.

---

#### `it('should render valid content without issues')`

**픽스처**: `text-1`(Text, `'Safe content'`) + `createBeginRendering('text-1')`.

**검증 항목**: `'Safe content'`가 DOM에 존재하며 예외 없이 렌더된다.

---

## 동작 흐름

1. 각 테스트는 `A2UIProvider` (또는 `TestWrapper`)로 트리를 감싸 컨텍스트를 제공한다.
2. 내부 렌더러 컴포넌트(또는 `TestRenderer`)가 `useA2UI()`의 `processMessages`를 통해 `ServerToClientMessage` 배열을 주입한다.
3. `processMessages`는 `createSurfaceUpdate`로 생성된 컴포넌트 맵과 `createBeginRendering`으로 지정된 루트 컴포넌트 ID를 처리하여 `A2UIRenderer`가 서피스를 렌더하도록 상태를 업데이트한다.
4. 비동기 단계 전환이 필요한 테스트는 `React.useState`로 단계를 추적하고 `setTimeout(..., 10)`으로 다음 단계를 트리거한 뒤 `waitFor`로 최종 DOM 상태를 단언한다.
5. 동기 테스트는 `render` 직후 `screen.getBy*`/`container.querySelector*`로 즉시 단언한다.
6. 에러 테스트는 `vi.spyOn`으로 콘솔을 억제하고 `expect(...).toThrow()`로 예외 전파를 확인한다.
