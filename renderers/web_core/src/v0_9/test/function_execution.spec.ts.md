# renderers/web_core/src/v0_9/test/function_execution.spec.ts

## 개요

`DataContext`의 함수 실행 기능을 검증하는 통합 테스트 파일이다. 특히 `subscribeDynamicValue`를 통해 스트리밍 함수(신호 반환)와 인수 변경에 반응하는 함수 호출이 올바르게 동작하는지 확인한다. 두 가지 테스트 케이스를 포함하며, 비동기 Promise 기반으로 실행된다.

## 의존성

### 외부 패키지
- `node:test` — `describe`, `it` (Node.js 내장 테스트 러너)
- `node:assert` — `assert` (Node.js 내장 어서션)
- `@preact/signals-core` — `signal` (반응형 신호 생성)

### 저장소 내부 모듈
- [`../state/data-model.ts`](../state/data-model.ts.md) — `DataModel`
- [`../rendering/data-context.ts`](../rendering/data-context.ts.md) — `DataContext`

## Exports

이 파일은 테스트 파일이므로 공개 export 없음.

## 상세 명세

### 비공개 헬퍼: `createTestDataContext`

**시그니처:**
```
createTestDataContext(model: DataModel, path: string, functionInvoker: any = () => null): DataContext
```

- `mockSurface` 객체를 익명 객체 리터럴로 구성한다. 필드는 `dataModel`(전달받은 `model`), `catalog`(`invoker` 프로퍼티에 `functionInvoker`를 담은 객체), `dispatchError`(빈 함수 `() => {}`)이다. 해당 객체는 `as any`로 캐스팅된다.
- 구성된 `mockSurface`와 `path`를 사용해 `new DataContext(mockSurface, path)`를 생성하고 반환한다.
- 목적: 실제 Surface 인프라 없이 `DataContext`를 격리 테스트할 수 있는 최소 환경 제공.

---

### 테스트 케이스 1: `'resolves and subscribes to metronome function'`

**검증하는 동작:**  
`subscribeDynamicValue`가 신호(`signal`)를 반환하는 함수(`metronome`)에 올바르게 구독하여, 신호 값의 순서대로 `'tick 0'`, `'tick 1'`, `'tick 2'`를 순서대로 수신하는지 확인한다.

**픽스처/모킹:**
- `DataModel` 인스턴스를 빈 상태로 생성.
- `functions: Map<string, Function>`에 `'metronome'` 키로 함수를 등록. 해당 함수는 `args['interval']`(기본값 100)을 interval로 사용해 `signal<string>('tick 0')`을 생성하고, `setInterval`로 매 틱마다 `subj.value = \`tick \${i++}\``를 업데이트하며, `abortSignal`의 `'abort'` 이벤트에서 `clearInterval`로 타이머를 정리한 뒤 신호 객체 `subj`를 반환한다.
- `functionInvoker`는 `(name, args, _ctx, abortSignal)` 형태로, `functions.get(name)`으로 함수를 조회해 있으면 호출하고 없으면 `undefined`를 반환한다.
- `createTestDataContext(dataModel, '/', functionInvoker)`로 `DataContext` 생성.

**동작 흐름:**
1. `dynamicValue`를 `{ call: 'metronome', args: { interval: 50 }, returnType: 'string' }`로 정의.
2. `new Promise<void>`를 반환하며, 내부에서 `context.subscribeDynamicValue<string>(dynamicValue, callback)`를 호출.
3. 콜백에서 `val`이 truthy이면 `values` 배열에 push. `values.length >= 3`이 되면 구독을 해제(`subscription.unsubscribe()`)하고, 순서대로 `assert.strictEqual`로 `values[0] === 'tick 0'`, `values[1] === 'tick 1'`, `values[2] === 'tick 2'`를 검증한다. 성공 시 `resolve()`, 실패 시 `reject(e)`.
4. 구독 직후 `subscription.value`가 truthy이면 `values`에 초기값을 즉시 push한다.

---

### 테스트 케이스 2: `'updates function output when arguments change'`

**검증하는 동작:**  
`subscribeDynamicValue`가 `DataModel`의 경로 참조(`{ path: '/msg' }`)를 인수로 사용하는 함수에 구독되어 있을 때, 데이터 모델의 값이 변경되면 함수가 재호출되어 새 출력을 수신하는지 확인한다.

**픽스처/모킹:**
- `DataModel` 인스턴스를 빈 상태로 생성.
- `functions: Map<string, Function>`에 `'echo'` 키로 함수를 등록. 이 함수는 `args['val']`을 받아 `` `echo: ${args['val']}` `` 문자열을 반환한다(동기 반환, 신호 아님).
- `functionInvoker`는 테스트 케이스 1과 동일한 구조.
- `createTestDataContext(dataModel, '/', functionInvoker)`로 `DataContext` 생성 후, `dataModel.set('/msg', 'hello')`로 초기값 설정.

**동작 흐름:**
1. `dynamicValue`를 `{ call: 'echo', args: { val: { path: '/msg' } }, returnType: 'string' }`로 정의. `args.val`이 경로 참조이므로 `DataContext`는 `/msg` 값을 해석해 전달한다.
2. `new Promise<void>`를 반환하며, `context.subscribeDynamicValue<string>(dynamicValue, callback)`를 호출.
3. 콜백에서 `val`이 truthy이면 `values`에 push. `values.length === 2`가 되면 구독 해제 후 `assert.strictEqual`로 `values[0] === 'echo: hello'`, `values[1] === 'echo: world'`를 검증한다.
4. 구독 직후 `subscription.value`가 truthy이면 즉시 `values`에 push.
5. `setTimeout(..., 50)` 후 `dataModel.set('/msg', 'world')`를 호출해 첫 번째 emit 이후 데이터를 변경, 두 번째 콜백 호출을 유도한다.

## 동작 흐름

파일 전체는 `describe('Function Execution in DataContext', ...)` 블록 아래 두 개의 비동기 테스트로 구성된다. 각 테스트는 독립적인 `DataModel`과 `DataContext`를 생성하고, `subscribeDynamicValue`를 사용해 함수 실행 결과를 구독한다. 테스트 1은 신호 기반 스트리밍 함수의 연속 emit을 검증하고, 테스트 2는 인수로 전달된 경로 참조가 변경될 때 함수 재실행을 검증한다. 두 테스트 모두 Promise를 반환해 Node.js 테스트 러너가 비동기 완료를 기다린다.
