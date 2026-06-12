# renderers/web_core/src/v0_9/rendering/data-context.ts

## 개요

`DataContext`는 A2UI 컴포넌트 트리에서 `DataModel`에 접근하기 위한 범위 지정(scoped) 인터페이스다. 컴포넌트가 `DataModel`과 직접 상호작용하는 대신 이 클래스를 사용하며, 절대/상대 JSON 포인터 경로의 해석, 리터럴·경로 바인딩·함수 호출을 포함하는 `DynamicValue`의 동기 및 반응형(reactive) 평가, 자식 스코프를 위한 중첩 컨텍스트 생성을 담당한다. 에러가 발생하면 `SurfaceModel`의 에러 디스패치 채널로 보고한다.

## 의존성

### 외부 패키지
- `@preact/signals-core` — `signal`, `computed`, `Signal`, `effect`
- `zod` — `z` (ZodError 식별에 사용)

### 저장소 내부 모듈
- [`../state/data-model.js`](../state/data-model.ts.md) — `DataModel`, `DataSubscription`
- [`../schema/common-types.js`](../schema/common-types.ts.md) — 타입 `DynamicValue`, `DataBinding`, `FunctionCall`, `Action`
- [`../errors.js`](../errors.ts.md) — `A2uiExpressionError`
- [`../catalog/types.js`](../catalog/types.ts.md) — `isSignal`
- [`../catalog/function_invoker.js`](../catalog/function_invoker.ts.md) — `FunctionInvoker`
- [`../state/surface-model.js`](../state/surface-model.ts.md) — `SurfaceModel`

## Exports

| 이름 | 종류 |
|------|------|
| `DataContext` | 클래스 |

## 상세 명세

### 클래스 `DataContext`

#### 필드

| 필드 | 타입 | 설명 |
|------|------|------|
| `surface` | `SurfaceModel<any>` | 생성자 매개변수, `readonly` |
| `path` | `string` | 이 컨텍스트의 현재 작업 디렉터리에 해당하는 절대 경로, `readonly` |
| `dataModel` | `DataModel` | `surface.dataModel`에서 가져온 공유 전역 인스턴스, `readonly` |
| `functionInvoker` | `FunctionInvoker` | `surface.catalog.invoker`에서 가져온 함수 호출 콜백, `readonly` |

#### 생성자 `constructor(surface: SurfaceModel<any>, path: string)`

`surface`와 `path`를 `readonly` 필드로 저장하고, `this.dataModel = surface.dataModel`, `this.functionInvoker = surface.catalog.invoker`를 설정한다.

#### getter `locale(): string | undefined`

`this.surface.locale`을 반환한다. 서피스로부터 로케일을 위임한다.

---

#### `set(path: string, value: any): void`

`resolvePath(path)`로 절대 경로를 구한 뒤 `this.dataModel.set(absolutePath, value)`를 호출한다. 컴포넌트가 사용자 입력 등의 상태 변경을 전역 모델에 반영할 때 사용하는 주 진입점이다.

---

#### `resolveDynamicValue<V>(value: DynamicValue): V`

`DynamicValue`를 **한 번만** 동기적으로 평가하여 런타임 값을 반환한다. 반응형 구독은 생성하지 않는다.

평가 순서:
1. **리터럴 검사**: `value`가 `null`이거나 `object`가 아니거나 배열이면 그대로 반환한다.
2. **경로 바인딩** (`'path' in value`): `resolvePath`로 절대 경로를 구한 뒤 `dataModel.get(absolutePath)`로 값을 읽는다.
3. **함수 호출** (`'call' in value`): 각 인수(`call.args`)를 재귀적으로 `resolveDynamicValue`로 해석한 뒤, `AbortController`를 생성하고 `evaluateFunctionReactive`를 호출한다. 결과가 `Signal`이면 `.peek()`으로 현재 값을 추출한다. 결과가 `undefined`면 그대로 반환한다.
4. 위 세 가지에 해당하지 않으면 `value`를 그대로 반환한다.

---

#### `subscribeDynamicValue<V>(value: DynamicValue, onChange: (value: V | undefined) => void): DataSubscription<V>`

`DynamicValue`를 반응형으로 평가하고, 값이 변경될 때마다 `onChange`를 호출한다. 반환값은 초기 값과 `unsubscribe` 메서드를 가진 `DataSubscription<V>`이다.

동작 단계:
1. `resolveSignal<V>(value)`로 신호(Signal)를 생성한다.
2. `isSync = true` 플래그를 세운 채 `effect(() => { ... })`를 등록한다. 이펙트 내부에서 `sig.value`를 읽어 `currentValue`를 갱신하고, `isSync`가 `false`인 경우에만 `onChange`를 호출한다.
3. 이펙트 등록 완료 후 즉시 `isSync = false`로 설정하여, 초기 동기 실행 시에는 `onChange`가 호출되지 않도록 한다.
4. 반환 객체의 `value` getter는 `currentValue`를 읽고, `unsubscribe`는 `dispose()`를 호출하고 신호에 `.unsubscribe()`가 있으면 추가로 호출한다.

---

#### `resolveSignal<V>(value: DynamicValue): Signal<V>`

`DynamicValue`를 완전히 반응형 Preact `Signal`로 변환한다.

평가 순서:
1. **리터럴**: `signal(value as V)`를 반환한다.
2. **경로 바인딩**: `resolvePath` 후 `dataModel.getSignal<V>(absolutePath)`를 반환한다.
3. **함수 호출 (인수 없음)**: `evaluateFunctionReactive`를 직접 호출하고, 결과가 `Signal`이 아니면 `signal(result)`로 감싼다. `AbortController.abort`를 신호의 `.unsubscribe` 속성으로 부착한다.
4. **함수 호출 (인수 있음)**: 각 인수를 재귀적으로 `resolveSignal`로 변환하여 `argSignals` 맵을 만든다. `computed()`로 인수 전체를 합산하는 `argsSig`를 만든다. `effect()`를 통해 `argsSig`가 바뀔 때마다 이전 구독과 `AbortController`를 정리하고 `evaluateFunctionReactive`를 재호출한다. 결과가 신호이면 내부 `effect`로 `resultSig`에 전파하고, 아니면 `resultSig.value`에 직접 할당한다. 에러 발생 시 `dispatchExpressionError`를 호출하고 `resultSig.value = undefined`로 리셋한다. `resultSig.unsubscribe`에 `stopper`, `innerUnsubscribe`, `abortController.abort`, 각 `argSig.unsubscribe` 호출을 묶어 놓는다.
5. 어떤 케이스도 해당하지 않으면 `signal(value as unknown as V)`를 반환한다.

---

#### `resolveAction(action: Action): any`

액션 객체를 한 단계(non-recursive) 해석한다.

- `'event' in action`인 경우: `action.event.context`의 각 값을 `resolveDynamicValue`로 해석하고, 원본 이벤트 객체에 해석된 컨텍스트를 병합하여 반환한다.
- `'functionCall' in action`인 경우: `resolveDynamicValue(action.functionCall)`을 반환한다.
- 그 외: `action`을 그대로 반환한다.

---

#### `private evaluateFunctionReactive<V>(name: string, args: Record<string, any>, abortSignal?: AbortSignal): Signal<V> | V`

`this.functionInvoker(name, args, this, abortSignal)`을 try/catch로 감싸 호출한다. 에러 발생 시 `dispatchExpressionError`를 호출하고 `undefined`를 반환한다.

---

#### `private dispatchExpressionError(e: any, name: string): void`

에러 종류에 따라 `EXPRESSION_ERROR` 코드의 에러 객체를 `surface.dispatchError`로 전송한다.

- `ZodError` (`e?.name === 'ZodError'` 또는 `e instanceof z.ZodError`): `A2uiExpressionError`로 래핑하고 `e.errors ?? e.issues`를 `details`로 사용한다.
- `A2uiExpressionError`: `e.expression`과 `e.details`를 그대로 사용한다.
- 일반 `Error`: `e.message`(없으면 기본 메시지)와 `{stack: e.stack}`을 `details`로 사용한다.

---

#### `nested(relativePath: string): DataContext`

`resolvePath(relativePath)`로 절대 경로를 계산하고, 동일한 `surface`와 새 절대 경로를 사용하는 새 `DataContext`를 반환한다. 리스트나 카드처럼 자식 스코프가 필요한 컴포넌트가 사용한다.

---

#### `private resolvePath(path: string): string`

상대 경로를 `this.path`를 기준으로 절대 경로로 변환한다.

- `path`가 `/`로 시작하면 절대 경로이므로 그대로 반환한다.
- `path`가 빈 문자열이거나 `'.'`이면 `this.path`를 반환한다.
- 그 외: `this.path`에서 끝의 `/`를 제거(단, 루트 `/` 제외)하고 `${base}/${path}` 형태로 결합하여 반환한다. `this.path`가 `'/'`이면 `base`를 `''`로 설정한다.

## 동작 흐름

1. `DataContext`는 생성 시 `SurfaceModel`로부터 `DataModel`과 `FunctionInvoker`를 추출하여 보유한다.
2. 컴포넌트가 데이터를 읽을 때는 `resolveDynamicValue` (1회성) 또는 `subscribeDynamicValue` (반응형)를 호출한다.
3. `subscribeDynamicValue`는 내부적으로 `resolveSignal`을 통해 Preact Signal 그래프를 구성하고, `effect`로 변경을 감지하여 `onChange` 콜백을 호출한다.
4. 함수 호출은 인수가 있을 경우 각 인수 신호를 `computed`로 집계하고, 인수 변경 시마다 함수를 재실행하는 반응형 체인을 형성한다.
5. 자식 컴포넌트는 `nested(relativePath)`로 더 깊은 경로에 스코핑된 자식 `DataContext`를 생성한다.
6. 에러는 throw하지 않고 `surface.dispatchError`로 보고하며, 반응형 신호는 `undefined`로 리셋된다.
