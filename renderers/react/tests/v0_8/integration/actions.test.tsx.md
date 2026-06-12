# renderers/react/tests/v0_8/integration/actions.test.tsx

## 개요

v0_8 컴포넌트에서 사용자 인터랙션이 발생했을 때 액션이 올바르게 `onAction` 콜백으로 전달되는지 검증하는 통합 테스트 파일이다. 버튼 클릭 이벤트, 액션 이름·컨텍스트 파라미터 전달, 데이터 모델 경로 바인딩 해석 등 액션 디스패치 전반을 커버한다. `fireEvent`로 DOM 이벤트를 발생시키고 `vi.fn()` 모킹으로 콜백 호출 내용을 검증한다.

## 의존성

### 외부 패키지
- `vitest` — `describe`, `it`, `expect`, `vi`
- `@testing-library/react` — `render`, `screen`, `fireEvent`
- `react` — JSX
- `@a2ui/web_core/types/types` — `Types.ServerToClientMessage`, `Types.A2UIClientEventMessage` (타입 전용 import)

### 저장소 내부 모듈
- [`../utils`](../utils/index.ts.md) — `TestWrapper`, `TestRenderer`, `createSurfaceUpdate`, `createBeginRendering`, `getMockCallArg` (v0_8 테스트 유틸리티 re-export)

## Exports

테스트 파일이므로 외부로 export하는 항목 없음.

## 테스트 케이스 명세

### `describe('Action Dispatch')`

모든 케이스에서 공통 패턴: `createSurfaceUpdate([...컴포넌트])` + `createBeginRendering(루트ID)`로 메시지 배열 구성 → `<TestWrapper onAction={mockOnAction}><TestRenderer messages={messages} /></TestWrapper>` 렌더링 → `fireEvent`로 이벤트 발생 → `expect(mockOnAction)` 단언.

| 테스트 케이스명 | 검증 동작 | 픽스처/모킹 |
|---|---|---|
| `should dispatch action with name` | 버튼 클릭 시 `mockOnAction`이 정확히 1회 호출되고, 첫 번째 인수의 `userAction.name`이 `'submit'`임 | `vi.fn()` mockOnAction. `Text(id='btn-text', usageHint='body')` + `Button(id='btn-1', child='btn-text', action.name='submit')`, `createBeginRendering('btn-1')`. `fireEvent.click(screen.getByRole('button'))`. `getMockCallArg<Types.A2UIClientEventMessage>(mockOnAction, 0)` |
| `should dispatch action with context parameters` | 버튼 클릭 시 `userAction.name`이 `'delete'`이고 `userAction.context`가 정의됨 | `Button` action에 `context: [{key:'itemId', value:{literalString:'item-123'}}, {key:'confirmed', value:{literalBoolean:true}}]` 설정. `createBeginRendering('btn-1')`. `getMockCallArg` index 0 |
| `should not call onAction if not provided` | `onAction` prop 없이 버튼 클릭해도 예외가 발생하지 않음 | `TestWrapper`에 `onAction` 미전달. `expect(() => fireEvent.click(screen.getByRole('button'))).not.toThrow()` |
| `should dispatch actions from different components` | 두 버튼 각각 클릭 시 1회/2회 호출되고, `getMockCallArg(mockOnAction, 0).userAction.name`이 `'action-1'`, `getMockCallArg(mockOnAction, 1).userAction.name`이 `'action-2'`임 | `Button(id='btn1', child='btn1-text', action.name='action-1')` + `Button(id='btn2', child='btn2-text', action.name='action-2')`를 `Column(id='col', children.explicitList=['btn1','btn2'])`으로 감쌈, `createBeginRendering('col')`. `fireEvent.click(screen.getByRole('button', {name:'Action 1'}))` 후 `screen.getByRole('button', {name:'Action 2'})` 클릭 |
| `should resolve path bindings in action context from data model` | 사용자가 `TextField`에 `'alice123'` 입력 후 버튼 클릭 시, 액션 컨텍스트 `username` 값이 데이터 모델에서 해석되어 `'alice123'`으로 전달됨. `event.userAction?.context`가 `{username: 'alice123'}`와 일치함 | `TextField(id='tf-1', text.path='form.username', label.literalString='Username')` + `Button(id='btn-1', child='btn-text', action.name='submit-form', context:[{key:'username', value.path:'form.username'}])` + `Column(id='col', children.explicitList=['tf-1','btn-1'])`, `createBeginRendering('col')`. `fireEvent.change(container.querySelector('input'), {target:{value:'alice123'}})` 후 Submit 버튼 클릭 |
| `should resolve mixed literal and path context parameters` | 리터럴(`formId`, `version`)과 경로 바인딩(`name`, `agreed`) 혼합 컨텍스트가 올바르게 해석되어 `{formId:'registration-form', version:2, name:'John Doe', agreed:true}` 전달됨 | `TextField(id='tf-name', text.path='form.name')` + `CheckBox(id='cb-agree', value.path='form.agreed')` + `Button(id='btn-1', child='btn-text', action.name='submit-form', context:[{key:'formId', value:{literalString:'registration-form'}}, {key:'version', value:{literalNumber:2}}, {key:'name', value:{path:'form.name'}}, {key:'agreed', value:{path:'form.agreed'}}])` + `Column(id='col', children.explicitList=['tf-name','cb-agree','btn-1'])`, `createBeginRendering('col')`. `fireEvent.change(container.querySelector('input[type="text"]'), {target:{value:'John Doe'}})` → `fireEvent.click(container.querySelector('input[type="checkbox"]'))` → Submit 버튼 클릭 |

## 동작 흐름

각 테스트는 서버 메시지를 구성하여 `TestRenderer`를 통해 A2UI 컴포넌트를 렌더링한다. `TestWrapper`의 `onAction` prop으로 전달된 `vi.fn()` 모킹이 `A2UIProvider` 컨텍스트 내부의 액션 디스패처에 연결된다. 사용자 이벤트(`fireEvent`)가 발생하면 컴포넌트 내부 이벤트 핸들러가 `userAction` 객체를 구성하여 `onAction` 콜백을 호출하며, `getMockCallArg`로 호출 인수를 타입 안전하게 꺼내 단언한다. 경로 바인딩이 있는 테스트는 `fireEvent.change`/`fireEvent.click`으로 데이터 모델을 먼저 업데이트한 뒤 Submit 버튼을 클릭해 해석 결과를 검증한다.
