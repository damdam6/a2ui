# renderers/web_core/src/v0_9/rendering/generic-binder.ts

## 개요

`GenericBinder`는 프레임워크에 독립적인 반응형 바인딩 엔진으로, A2UI의 raw JSON 컴포넌트 설정을 Zod 스키마와 결합하여 완전히 해석된 props 스트림으로 변환한다. Zod 스키마를 정적으로 분석(`scrapeSchemaBehavior`)하여 각 필드의 동작 유형(동적 바인딩, 액션, 구조적 자식 목록, 유효성 검사 등)을 파악하고, 이를 기반으로 데이터 구독·구독 해제를 자동으로 관리한다. React, Angular 등 프레임워크별 어댑터는 `subscribe()` 인터페이스를 통해 props 변경을 수신한다.

## 의존성

### 외부 패키지
- `zod` (`z`)

### 저장소 내부 모듈
- [`./component-context.js`](./component-context.ts.md) — `ComponentContext` 타입 (컴포넌트 모델, 데이터 컨텍스트, 액션 디스패처 포함)
- [`../schema/common-types.js`](../schema/common-types.ts.md) — `Action`, `ChildList`, `DataBinding`, `FunctionCall` 타입

## Exports

| 이름 | 종류 | 설명 |
|------|------|------|
| `BehaviorNode` | 타입 (유니온) | 스키마 필드의 런타임 동작 유형 분류 |
| `scrapeSchemaBehavior` | 함수 | Zod 스키마 트리를 `BehaviorNode` 맵으로 변환 |
| `ResolveA2uiProp<T>` | 제네릭 타입 | raw Zod 추론 타입을 런타임 등가물로 매핑 |
| `GenerateSetters<T>` | 제네릭 타입 | 동적 프로퍼티에 대한 setter 메서드 시그니처 자동 생성 |
| `ResolveA2uiProps<T>` | 제네릭 타입 | `GenericBinder`의 최종 출력 타입 |
| `GenericBinder<T>` | 클래스 | 반응형 props 바인딩 엔진 |

## 상세 명세

### 타입: `BehaviorNode`

Zod 스키마에서 파싱된 프로퍼티의 의도된 런타임 동작을 나타내는 유니온 타입이다.

- `{type: 'DYNAMIC'}` — `DataModel`에 바인딩 가능한 동적 값 (예: `DynamicString`). Binder가 데이터 변경을 구독하고 원시값을 방출한다.
- `{type: 'ACTION'}` — 사용자 인터랙션을 나타내는 프로퍼티 (예: `Action`). Binder가 페이로드 바인딩을 깊이 해석하고 호출 가능한 `() => void` 클로저를 반환한다.
- `{type: 'STRUCTURAL'}` — 자식 컴포넌트 렌더링을 지정하는 프로퍼티 (예: `ChildList`). Binder가 `{ id, basePath }` 형태의 객체 목록을 출력한다.
- `{type: 'CHECKABLE'}` — 유효성 검사 배열을 처리하는 특수 프로퍼티 (`checks`). Binder가 규칙을 반응형으로 평가하고 부모 객체에 `isValid`와 `validationErrors`를 주입한다.
- `{type: 'STATIC'}` — 반응형 구독이나 해석이 필요 없는 원시값.
- `{type: 'OBJECT'; shape: Record<string, BehaviorNode>}` — 중첩 스키마를 재귀적으로 순회하는 노드.
- `{type: 'ARRAY'; element: BehaviorNode}` — 배열 요소를 재귀적으로 처리하는 노드.

### 함수: `scrapeSchemaBehavior(schema: z.ZodTypeAny): BehaviorNode`

외부에 노출되는 진입점. 내부적으로 `getFieldBehavior(schema)`를 호출하여 결과를 반환한다.

### 함수: `getFieldBehavior(type: z.ZodTypeAny, propertyName?: string): BehaviorNode` (비공개)

Zod 타입 하나를 받아 `BehaviorNode`를 결정한다.

1. `ZodOptional`, `ZodNullable`, `ZodDefault` 래퍼를 모두 벗겨 내부 타입을 얻는다 (`current._def.innerType`으로 순회).
2. `propertyName === 'checks'`이면 즉시 `{type: 'CHECKABLE'}`을 반환한다.
3. 타입명이 `ZodUnion`이면 `options` 배열을 검사하여 A2UI 프리미티브를 식별한다:
   - `{ event: ... }` 형태의 `ZodObject`를 포함하면 → `ACTION`
   - `{ path: ... }` 는 있지만 `{ componentId: ... }`가 없는 `ZodObject`를 포함하면 → `DYNAMIC`
   - `{ componentId: ... }`와 `{ path: ... }` 모두 있는 `ZodObject`를 포함하면 → `STRUCTURAL`
4. `ZodArray`이면 `{type: 'ARRAY', element: getFieldBehavior(요소 타입)}`을 재귀 반환한다.
5. `ZodObject`이면 `shape()`의 모든 키에 대해 재귀 호출하여 `{type: 'OBJECT', shape: {...}}`을 반환한다. 키가 `checks`이면 해당 키의 결과는 `CHECKABLE`이 된다.
6. 어떤 경우에도 해당하지 않으면 `{type: 'STATIC'}`을 반환한다.

### 타입: `DynamicTypes` (비공개)

`DataBinding | FunctionCall` — 동적으로 해석이 필요한 타입의 조합.

### 타입: `IsDynamic<T>` (비공개)

`DataBinding extends NonNullable<T> ? true : false` — 타입 `T`에서 `DataBinding`을 포함하는지 여부를 컴파일 타임에 판단하는 조건부 타입.

### 타입: `ResolveA2uiProp<T>`

raw Zod 추론 타입을 런타임 등가물로 매핑하는 조건부 타입이다.

- `T`의 Non-Nullable 부분이 `Action`을 확장하면 → `(() => void) | Extract<T, undefined>`
- `T`의 Non-Nullable 부분이 `ChildList`를 확장하면 → `any | Extract<T, undefined>`
- `T`에서 `DynamicTypes`를 제외한 나머지가 `never`이면 (완전히 동적) → `any`
- 그 외 → `Exclude<T, DynamicTypes>` (동적 타입을 제외한 구체적 타입)

### 타입: `GenerateSetters<T>`

`T`의 모든 키 `K`에 대해, `IsDynamic<T[K]>`가 `true`인 키에 한해 `set${Capitalize<K>}` 이름의 setter 메서드를 생성한다. setter 시그니처는 `(value: Exclude<NonNullable<T[K]>, DynamicTypes>) => void`.

### 타입: `ResolveA2uiProps<T>`

`GenericBinder`의 최종 출력 타입. `T`의 각 키를 `ResolveA2uiProp`으로 해석한 결과와 `GenerateSetters<T>`를 교차(intersection)하고, 선택적으로 `isValid?: boolean`과 `validationErrors?: string[]`을 추가한다.

### 클래스: `GenericBinder<T>`

#### 필드 (모두 private, 별도 명시 제외)

- `dataListeners: (() => void)[]` — 현재 구독 중인 데이터 리스너들의 unsubscribe 함수 목록.
- `propsListeners: ((props: T) => void)[]` — props 변경을 기다리는 외부 리스너 목록.
- `currentProps: Partial<T>` (public) — 현재 해석된 props의 최신 스냅샷.
- `compUnsub?: () => void` — 컴포넌트 모델 업데이트 구독 해제 함수.
- `isConnected: boolean` — 활성 구독 상태 여부.
- `context: ComponentContext` — 원본 JSON 설정, 데이터 컨텍스트, 액션 디스패처.
- `behaviorTree: BehaviorNode` — 스키마 분석 결과 트리.

#### 생성자: `constructor(context: ComponentContext, schema: z.ZodTypeAny)`

1. `context`와 `behaviorTree`를 초기화한다.
2. `scrapeSchemaBehavior(schema)`로 동작 트리를 생성한다.
3. 루트 동작 타입이 `OBJECT`가 아니면 빈 `{type: 'OBJECT', shape: {}}`로 대체한다.
4. `resolveInitialProps()`를 호출해 초기 props를 동기적으로 해석한다.

#### 메서드: `resolveInitialProps()` (private)

`componentModel.properties`를 읽고 `resolveAndBind(..., true)`(동기 모드)를 호출하여 `currentProps`에 초기값을 설정한다. 동기 모드에서는 구독이 즉시 해제된다.

#### 메서드: `connect()` (private)

이미 연결된 경우(`isConnected === true`) 반환한다. 그렇지 않으면:
1. `isConnected`를 `true`로 설정.
2. `componentModel.onUpdated`를 구독하여 컴포넌트 JSON이 변경될 때 `rebuildAllBindings()`가 호출되도록 한다.
3. `compUnsub`에 해제 함수를 저장.
4. `rebuildAllBindings()`를 즉시 호출.

#### 메서드: `rebuildAllBindings()` (private)

1. 기존 모든 `dataListeners`를 호출하여 데이터 구독을 해제하고 배열을 비운다.
2. 최신 `componentModel.properties`를 읽고 `resolveAndBind(..., false)`(비동기 모드)로 새 props를 구성한다.
3. `currentProps`를 업데이트하고 `notify()`를 호출한다.

#### 메서드: `resolveAndBind(value, behavior, path, isSync): any` (private)

`value`가 `null`/`undefined`이면 그대로 반환한다. `behavior.type`에 따라 다음을 수행한다:

- **`DYNAMIC`**: `dataContext.subscribeDynamicValue(value, callback)`를 호출한다. 콜백에서는 `updateDeepValue(path, newVal)`와 `notify()`를 호출한다. `isSync`가 `false`이면 unsubscribe 함수를 `dataListeners`에 등록하고, `true`이면 즉시 해제한다. `bound.value`(현재 해석된 원시값)를 반환한다.

- **`ACTION`**: `() => void` 클로저를 반환한다. 실행 시 내부에서 `resolveDeepSync` 헬퍼로 페이로드의 모든 `{ path }` 또는 `{ call }` 객체를 재귀적으로 해석한 뒤 `context.dispatchAction(resolved)`를 호출한다.

- **`STRUCTURAL`**: `value`가 `{ path, componentId }` 형태의 객체인 경우, `dataContext.subscribeDynamicValue({path}, callback)`를 구독한다. 콜백 및 즉시 처리 시, 해당 path의 배열 데이터를 읽고 각 인덱스에 대해 `{ id: value.componentId, basePath: listContext.nested(i).path }` 형태의 객체 배열을 만들어 `updateDeepValue`로 저장한다. 그렇지 않으면 `value`를 그대로 반환한다.

- **`CHECKABLE`**: `value`가 배열인 경우 각 규칙에 대해 `ruleResults` 배열을 유지한다. 각 규칙의 `condition`(또는 규칙 자체)을 `subscribeDynamicValue`로 구독하고, 평가 결과가 변경될 때마다 `updateDeepValue`로 부모 경로에 `isValid`와 `validationErrors`를 주입하고 `notify()`를 호출한다. `checks` 프로퍼티 자체는 원본 규칙 배열을 그대로 반환한다. 초기 상태도 동기적으로 설정한다.

- **`STATIC`**: `value`를 그대로 반환한다.

- **`ARRAY`**: `value`가 배열인 경우 각 요소에 대해 `resolveAndBind(item, behavior.element, [...path, index], isSync)`를 재귀 호출하여 새 배열을 반환한다.

- **`OBJECT`**: 두 단계로 처리한다.
  1. `value`의 모든 키에 대해 `behavior.shape[k]`(없으면 `STATIC`)을 사용해 재귀 호출한다.
  2. `behavior.shape`의 모든 `DYNAMIC` 키에 대해 setter 함수 `set${K}`를 `result`에 추가한다. setter는 `rawPropValue`가 `{ path }` 형태이면 `dataContext.set(path, newValue)`를 호출한다.

#### 메서드: `updateDeepValue(path: string[], newValue: any)` (private)

`cloneAndUpdate(currentProps, path, newValue)`를 호출하여 `currentProps`를 불변 방식으로 업데이트한다.

#### 메서드: `cloneAndUpdate(obj, path, newValue): any` (private)

재귀적 불변 업데이트 헬퍼. `path`가 비어 있으면 `newValue`를 반환한다. `obj`가 배열이면 얕은 복사 후 `Number(path[0])` 인덱스를 재귀 업데이트한다. 객체이면 스프레드로 얕은 복사 후 해당 키를 재귀 업데이트한다.

#### 메서드: `dispose()`

`isConnected`가 `false`이면 반환. 그렇지 않으면 모든 `dataListeners`를 호출해 구독을 해제하고 배열을 비운다. `compUnsub`을 호출하고 `undefined`로 초기화한다. `isConnected`를 `false`로 설정한다.

#### 메서드: `notify()` (private)

모든 `propsListeners`에 `currentProps as T`를 전달하며 호출한다.

#### 메서드: `subscribe(listener: (props: T) => void)`

`propsListeners`가 비어 있으면 `connect()`를 호출한다(지연 연결). `listener`를 `propsListeners`에 추가한다. `{ unsubscribe }` 객체를 반환하며, `unsubscribe()`는 `propsListeners`에서 해당 리스너를 제거하고, 목록이 비게 되면 `dispose()`를 호출한다.

#### getter: `snapshot`

`currentProps as T`를 반환한다.

## 동작 흐름

1. **초기화**: 생성자에서 스키마를 분석하고 동기 모드로 초기 props를 계산하여 `currentProps`에 저장한다.
2. **지연 활성화**: 첫 번째 `subscribe()` 호출 시 `connect()`가 실행되어 컴포넌트 모델 변경 구독을 설정한다.
3. **데이터 반응성**: `DYNAMIC`, `STRUCTURAL`, `CHECKABLE` 필드는 데이터 경로를 구독한다. 데이터가 변경되면 `updateDeepValue` → `notify()` 경로로 변경이 전파된다.
4. **구조적 변경**: `componentModel.onUpdated`가 발생하면 `rebuildAllBindings()`가 기존 구독을 모두 해제하고 새 구독을 설정한다.
5. **정리**: 마지막 `propsListeners`가 제거되거나 `dispose()`가 직접 호출되면 모든 구독이 해제된다.
