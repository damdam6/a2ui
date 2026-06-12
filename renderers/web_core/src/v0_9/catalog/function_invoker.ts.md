# renderers/web_core/src/v0_9/catalog/function_invoker.ts

## 개요

카탈로그 함수를 이름으로 호출하는 콜백의 타입 계약을 단독으로 정의하는 파일이다. `FunctionInvoker` 타입 하나만 선언하며, 실제 구현 로직은 포함하지 않는다. 이 타입은 `Catalog` 클래스의 `invoker` 필드 타입으로 사용되고, `DataContext`에 전달된다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈

- [`../rendering/data-context.js`](../rendering/data-context.ts.md) — `DataContext` 타입

## Exports

| 이름 | 종류 | 설명 |
|---|---|---|
| `FunctionInvoker` | 타입(함수 시그니처) | 카탈로그 함수를 이름으로 호출하는 콜백 타입 |

## 상세 명세

### `FunctionInvoker`

```
type FunctionInvoker = (
  name: string,
  args: Record<string, any>,
  context: DataContext,
  abortSignal?: AbortSignal,
) => any;
```

| 매개변수 | 타입 | 설명 |
|---|---|---|
| `name` | `string` | 호출할 함수 이름 |
| `args` | `Record<string, any>` | 함수에 전달할 인수 맵 |
| `context` | `DataContext` | 함수가 실행되는 데이터 컨텍스트 |
| `abortSignal` | `AbortSignal` (optional) | 비동기/장기 실행 작업 취소 신호 |

반환 타입은 `any`로, 리터럴 값, Preact `Signal`, 또는 `Promise`가 될 수 있다. 호출자가 반환값의 종류를 판단해 처리한다.

## 동작 흐름

이 파일은 타입 정의만 포함하므로 런타임 동작이 없다. 구체적인 `FunctionInvoker` 구현은 `Catalog` 클래스의 생성자 내부에서 `this.invoker`로 할당된다.
