# renderers/react/tests/v0_8/integration/templates.test.tsx

## 개요

A2UI v0.8 렌더러의 **템플릿 확장(template expansion)** 기능에 대한 통합 테스트 파일이다.
데이터 바인딩을 통한 배열·Map 반복 렌더링, 중첩 템플릿, 늦게 도착하는 데이터 처리, 데이터 변경 시 재렌더링 등을 실제 `A2UIProvider` + `A2UIRenderer` 조합으로 검증한다.
각 테스트는 별도의 인라인 React 컴포넌트(`useA2UI()` + `useEffect`)를 정의해 메시지를 주입하는 패턴을 공유한다.

## 의존성

### 외부 패키지
- `vitest` — `describe`, `it`, `expect`, `vi`
- `@testing-library/react` — `render`, `screen`, `fireEvent`, `waitFor`
- `react` — `React`, `useEffect`
- `@a2ui/web_core/types/types` — 타입 전용 import (`Types` 네임스페이스)

### 저장소 내부 모듈
- [`../../../src/v0_8`](../../../src/v0_8.md) — `A2UIProvider`, `A2UIRenderer`, `useA2UI`
- [`../utils`](../utils/index.ts.md) — `createSurfaceUpdate`, `createBeginRendering`, `createDataModelUpdateSpec`, `getMockCallArg`, `getElement`

## Exports

없음 (테스트 파일이므로 외부로 내보내는 항목 없음).

## 테스트 케이스 상세

### describe: `Template Integration`

모든 케이스는 공통적으로 다음 패턴을 따른다.
1. 인라인 컴포넌트 내 `useA2UI()`로 `processMessages`를 획득한다.
2. `useEffect` 안에서 `processMessages([...])` 호출로 `createDataModelUpdateSpec`, `createSurfaceUpdate`, `createBeginRendering` 메시지를 순서대로 전달한다.
3. `<A2UIProvider>` + `<A2UIRenderer surfaceId="@default" />`로 렌더링한다.
4. `screen` 또는 `container` DOM 쿼리로 결과를 단언한다.

---

#### `should expand template with dataBinding to array`

- **컴포넌트**: `TemplateRenderer`
- **픽스처**: `items` 키에 `[{name:'Item A'},{name:'Item B'},{name:'Item C'}]` JSON 배열 저장.
- **구조**: `List` → `template` (dataBinding: `'/items'`, componentId: `'item-template'`) → `Text` (path: `'name'`).
- **검증**: `'Item A'`, `'Item B'`, `'Item C'` 각각 DOM에 존재.

---

#### `should handle template with primitive array values using path "."`

- **컴포넌트**: `PrimitiveTemplateRenderer`
- **픽스처**: `tags` 키에 `['travel','paris','guide']` 원시값 배열.
- **구조**: `Row` → `template` (dataBinding: `'/tags'`) → `Text` (path: `'.'`) — 현재 아이템 자체를 참조.
- **검증**: `'travel'`, `'paris'`, `'guide'` 모두 DOM에 존재.

---

#### `should expand nested templates with layered data contexts`

- **컴포넌트**: `NestedTemplateRenderer`
- **픽스처**: `days` 키에 `[{title:'Day 1', activities:['Morning Walk','Museum Visit']},{title:'Day 2', activities:['Market Trip']}]`.
- **구조**:
  - `List` → `template` (dataBinding: `'/days'`, componentId: `'day-template'`)
  - `day-template`: `Column` → `['day-title', 'activity-list']`
  - `day-title`: `Text` (path: `'title'`)
  - `activity-list`: `List` → `template` (dataBinding: `'activities'`, componentId: `'activity-template'`)
  - `activity-template`: `Text` (path: `'.'`)
- **중첩 컨텍스트**: 외부 루프에서 확립된 데이터 컨텍스트(day 객체) 위에 내부 루프가 `activities` 배열을 추가로 바인딩한다.
- **검증**: `'Day 1'`, `'Day 2'`, `'Morning Walk'`, `'Museum Visit'`, `'Market Trip'` 전부 존재.

---

#### `should rebuild template when data arrives after components`

- **컴포넌트**: `LateDataRenderer` (`React.useState(0)` 단계 제어)
- **픽스처·타이밍**:
  - step 0: 컴포넌트 정의(`createSurfaceUpdate`) + `createBeginRendering` 전송, `setStep(1)` 호출.
  - step 1: 10ms 지연(`setTimeout`) 후 `items: [{name:'Late Item 1'},{name:'Late Item 2'}]` 데이터 전송.
- **검증**: `waitFor` 안에서 `'Late Item 1'`, `'Late Item 2'` 존재 확인. (데이터가 컴포넌트 등록 이후에 도착해도 정상 렌더링됨을 검증.)

---

#### `should expand template with dataBinding to Map (from valueMap)`

- **컴포넌트**: `MapTemplateRenderer`
- **픽스처**: `users` 키에 `[{name:'Alice',role:'Admin'},{name:'Bob',role:'User'}]`.
  - 주석에 따르면 nested `valueMap`은 현재 Zod 스키마에서 표현 불가하므로 JSON 인코딩된 배열을 사용.
- **구조**: `List` → `template` (dataBinding: `'/users'`, componentId: `'user-card'`) → `Card` → `Column` → `['user-name','user-role']` (Text path: `'name'`, `'role'`).
- **검증**: `'Alice'`, `'Admin'`, `'Bob'`, `'User'` 모두 존재.

---

#### `should handle template with complex nested components`

- **픽스처**: `products: [{id:'prod-1',name:'Widget',price:'$10'},{id:'prod-2',name:'Gadget',price:'$20'}]`.
- **구조**: `List` → `template` (dataBinding: `'/products'`, componentId: `'product-row'`) → `Row` → `['product-name','product-price','buy-button']`.
  - `buy-button`: `Button` (child: `'buy-text'`, action: `{name:'buy', context:[{key:'productId', value:{path:'id'}}]}`).
- **모킹**: `vi.fn()`으로 만든 `mockOnAction`을 `<A2UIProvider onAction={mockOnAction}>` 에 전달.
- **검증**:
  - `'Widget'`, `'$10'`, `'Gadget'`, `'$20'` 텍스트 존재.
  - `role='button'` name `'Buy'` 버튼이 정확히 2개.
  - 첫 번째 버튼 클릭 시 `mockOnAction` 1회 호출, `event.userAction.name === 'buy'` 확인.
  - `getElement(buyButtons, 0)`으로 첫 번째 버튼 요소 추출 후 `fireEvent.click`.
  - `getMockCallArg<Types.A2UIClientEventMessage>(mockOnAction, 0)`로 첫 번째 호출 인자 타입 검증.

---

#### `should handle empty data array gracefully`

- **픽스처**: `items: []` (빈 배열).
- **구조**: `Column` → `['header','item-list']`. `header`: `Text` literalString `'Items:'`. `item-list`: `List` → template.
- **검증**: 예외 없이 렌더링되며 `'Items:'` 텍스트가 DOM에 존재한다. (빈 배열에 대한 오류 없는 처리 확인.)

---

#### `should update template when data changes`

- **컴포넌트**: `DataChangeRenderer` (단계 제어)
- **픽스처·타이밍**:
  - step 0: `items: [{name:'Original'}]` + 컴포넌트 구조 전송, `setStep(1)`.
  - step 1: 10ms 후 `items: [{name:'Updated 1'},{name:'Updated 2'}]` 로 갱신.
- **검증**: 초기 `'Original'` 존재 → `waitFor`로 `'Updated 1'`, `'Updated 2'` 존재 + `'Original'` 소멸 확인.

---

## 동작 흐름

각 테스트는 독립적인 React 트리를 구성한다. `A2UIProvider`가 상태 컨텍스트를 제공하고, 인라인 컴포넌트가 `useEffect` 내에서 `processMessages`를 통해 서버 메시지(데이터 모델 업데이트 → 서피스 업데이트 → 렌더링 시작 순)를 주입한다. `A2UIRenderer`는 주입된 상태를 기반으로 DOM을 생성하며, 비동기 케이스는 `waitFor`로 상태 안정화를 대기한다.
