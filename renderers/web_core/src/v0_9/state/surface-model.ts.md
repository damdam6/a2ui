# renderers/web_core/src/v0_9/state/surface-model.ts

## 개요

단일 UI Surface의 상태 모델이다. 하나의 Surface는 컴포넌트 집합(`SurfaceComponentsModel`)과 클라이언트 데이터(`DataModel`)를 소유하며, 컴포넌트에서 발생한 이벤트를 `A2uiClientAction`으로 변환·검증하여 상위 계층에 발행한다. 또한 Surface 레벨의 에러를 `onError` 스트림으로 발행하는 역할도 담당한다.

## 의존성

### 저장소 내부 모듈

- [`./data-model.ts`](./data-model.ts.md) — `DataModel`
- [`../catalog/types.ts`](../catalog/types.ts.md) — `Catalog`, `ComponentApi`
- [`./surface-components-model.ts`](./surface-components-model.ts.md) — `SurfaceComponentsModel`
- [`../common/events.ts`](../common/events.ts.md) — `EventEmitter`, `EventSource`
- [`../schema/client-to-server.ts`](../schema/client-to-server.ts.md) — `A2uiClientAction`, `A2uiClientActionSchema`

### 외부 패키지

없음

## Exports

| 이름 | 종류 |
|---|---|
| `ActionListener` | 타입 별칭 |
| `SurfaceModel<T>` | 클래스 (제네릭) |

## 상세 명세

### 타입 `ActionListener`

```
type ActionListener = (action: A2uiClientAction) => void | Promise<void>
```

Surface에서 발행되는 액션을 수신하는 함수의 타입. 동기/비동기 모두 허용.

### 클래스 `SurfaceModel<T extends ComponentApi = ComponentApi>`

#### 필드

| 필드 | 타입 | 접근 | 설명 |
|---|---|---|---|
| `id` | `string` | `readonly` | Surface 고유 식별자 |
| `catalog` | `Catalog<T>` | `readonly` | 이 Surface가 사용하는 컴포넌트 카탈로그 |
| `theme` | `any` | `readonly` | Surface에 적용할 테마 객체. 기본값 `{}` |
| `sendDataModel` | `boolean` | `readonly` | 클라이언트가 전체 데이터 모델을 서버로 전송할지 여부. 기본값 `false` |
| `locale` | `string \| undefined` | `readonly` | 로케일 감지 함수에서 사용할 로케일 문자열. 선택적 |
| `dataModel` | `DataModel` | `readonly` | 이 Surface의 반응형 데이터 저장소 |
| `componentsModel` | `SurfaceComponentsModel` | `readonly` | 이 Surface의 컴포넌트 컬렉션 |
| `_onAction` | `EventEmitter<A2uiClientAction>` | `private readonly` | 액션 이벤트 발행기 |
| `_onError` | `EventEmitter<any>` | `private readonly` | 에러 이벤트 발행기 |
| `onAction` | `EventSource<A2uiClientAction>` | `readonly` | 외부 구독 전용 액션 스트림 |
| `onError` | `EventSource<any>` | `readonly` | 외부 구독 전용 에러 스트림 |

#### 생성자

```
constructor(
  id: string,
  catalog: Catalog<T>,
  theme: any = {},
  sendDataModel: boolean = false,
  locale?: string
)
```

`readonly` 필드들을 설정한 뒤 `this.dataModel = new DataModel({})`와 `this.componentsModel = new SurfaceComponentsModel()`으로 하위 모델을 초기화한다. 초기 데이터는 빈 객체 `{}`이다.

#### `dispatchAction(payload: any, sourceComponentId: string): Promise<void>`

컴포넌트에서 이벤트가 발생할 때 호출되어 서버로 전송할 액션을 구성·검증·발행한다.

1. `payload`가 객체이고 `event` 프로퍼티가 있으며 truthy인지 검사한다.
2. 다음 구조의 객체를 구성한다:
   - `name`: `payload.event.name`
   - `surfaceId`: `this.id`
   - `sourceComponentId`: 인자로 전달된 값
   - `timestamp`: `new Date().toISOString()`
   - `context`: `payload.event.context || {}`
3. `A2uiClientActionSchema.safeParse(actionToValidate)`로 Zod 검증을 수행한다.
4. 검증 성공 시 `await this._onAction.emit(validationResult.data)`.
5. 실패 시 `console.error`로 포맷된 오류를 출력하고 발행하지 않는다.
6. `functionCall` 타입의 로컬 액션은 렌더러 또는 바인더가 처리하므로 이 메서드에서는 다루지 않는다.

#### `dispatchError(error: {code: string; message: string; [key: string]: any}): Promise<void>`

`error` 객체에 `surfaceId: this.id`를 추가한 객체를 `await this._onError.emit(...)` 으로 발행한다.

#### `dispose(): void`

다음 순서로 정리한다:
1. `this.dataModel.dispose()` — Signal·구독 정리
2. `this.componentsModel.dispose()` — 컴포넌트 컬렉션 및 이벤트 정리
3. `this._onAction.dispose()` — 액션 리스너 제거
4. `this._onError.dispose()` — 에러 리스너 제거

## 동작 흐름

`SurfaceModel`은 단일 Surface 범위 안에서 데이터와 컴포넌트를 소유하고, 렌더러로부터 전달된 이벤트 payload를 표준 `A2uiClientAction`으로 변환해 외부에 발행하는 중간 계층이다. `SurfaceGroupModel`이 여러 `SurfaceModel`을 소유하며, 각 Surface의 `onAction`을 구독하여 모든 Surface의 액션을 단일 스트림으로 집약한다. Surface가 닫히거나 연결이 종료될 때 `dispose()`를 호출하면 소유한 데이터·컴포넌트·이벤트 리스너가 모두 정리된다.
