# renderers/angular/src/v0_8/data/processor.ts

## 개요

a2ui의 핵심 메시지 처리 서비스로, `@a2ui/web_core`의 `A2uiMessageProcessor`를 Angular DI 환경에 통합한다. 서버에서 수신한 메시지를 처리하고, 사용자 액션을 Observable 이벤트 스트림으로 발행하며, Angular 변경 감지와 연동하기 위한 버전 신호(version signal)를 관리한다. 또한 렌더링 준비가 완료된 표면(surface)만 필터링하여 반환하는 역할을 담당한다.

## 의존성

### 외부 패키지
- `@angular/core` — `Injectable`, `signal`
- `rxjs` — `Subject`, `Observable`
- `@a2ui/web_core/v0_8` — `A2uiMessageProcessor`, `Surface`, `ServerToClientMessage`, `AnyComponentNode` (as `WebCore`)

### 저장소 내부 모듈
- [`../types`](../types.ts.md) — `A2UIClientEventMessage`, `AnyComponentNode`, `ServerToClientMessage` (타입 전용)

## Exports

| 이름 | 종류 |
|---|---|
| `A2UIClientEvent` | 인터페이스 |
| `DispatchedEvent` | 타입 별칭 |
| `MessageProcessor` | Angular injectable 클래스 |

## 상세 명세

### `A2UIClientEvent` 인터페이스

| 필드 | 타입 | 설명 |
|---|---|---|
| `message` | `A2UIClientEventMessage` | 발송된 클라이언트 이벤트 메시지 |
| `completion` | `Subject<ServerToClientMessage[]>` | 서버 응답을 전달하여 dispatch Promise를 해결하는 Subject |

---

### `DispatchedEvent` 타입 별칭

`A2UIClientEvent`의 별칭. 하위 호환성 또는 명시적 의미 부여를 위해 존재한다.

---

### `MessageProcessor` 클래스

`@Injectable({ providedIn: 'root' })` 데코레이터를 가진 서비스 클래스.

#### 필드

| 이름 | 타입 | 접근 | 설명 |
|---|---|---|---|
| `baseProcessor` | `WebCore.A2uiMessageProcessor` | private | 실제 메시지 처리 위임 대상 |
| `eventsSubject` | `Subject<A2UIClientEvent>` | private readonly | 이벤트를 발행하는 내부 Subject |
| `events` | `Observable<A2UIClientEvent>` | readonly | `eventsSubject`의 공개 Observable |
| `versionSignal` | `Signal<number>` (writable) | private readonly | 표면 데이터 변경을 Angular에 알리는 카운터 신호 |
| `version` | `Signal<number>` (readonly) | readonly | `versionSignal`의 읽기 전용 노출 |

#### `constructor()`

`new WebCore.A2uiMessageProcessor()`로 `baseProcessor`를 초기화한다.

---

#### `notify(): void` (private)

`versionSignal.update(v => v + 1)`으로 버전 카운터를 증가시킨다. 표면 데이터가 변경될 때마다 호출하여 Angular의 신호 기반 반응성을 트리거한다.

---

#### `processMessages(messages: ServerToClientMessage[]): void`

1. `this.baseProcessor.processMessages(messages as WebCore.ServerToClientMessage[])`를 호출하여 서버 메시지를 처리.
2. `this.notify()`를 호출하여 변경 사항을 Angular에 알린다.

---

#### `dispatch(message: A2UIClientEventMessage): Promise<ServerToClientMessage[]>`

1. 새 `Subject<ServerToClientMessage[]>` (`completion`)을 생성.
2. `completion`을 구독하는 `Promise`를 생성: `next` 시 `resolve`, `error` 시 `reject`.
3. `{message, completion}`을 `eventsSubject`에 발행.
4. Promise를 반환하여 호출자가 서버 응답을 `await`할 수 있게 한다.

---

#### `getData(node: AnyComponentNode, path: string, surfaceId?: string | null): unknown`

`this.baseProcessor.getData(node, path, surfaceId ?? undefined)`에 위임하여 데이터 모델 값을 반환한다.

---

#### `setData(node: AnyComponentNode | null, path: string, value: any, surfaceId: string): void`

1. `this.baseProcessor.setData(node, path, value, surfaceId)`에 위임.
2. `this.notify()`를 호출하여 변경 사항을 Angular에 알린다.

---

#### `resolvePath(path: string, dataContextPath?: string): string`

`this.baseProcessor.resolvePath(path, dataContextPath)`에 위임하여 해석된 경로 문자열을 반환한다.

---

#### `getSurfaces(): ReadonlyMap<string, WebCore.Surface>`

1. `this.versionSignal()`을 읽어 Angular 신호 의존성을 등록한다(변경 시 재평가 트리거).
2. `this.baseProcessor.getSurfaces()`로 전체 표면 맵을 가져온다.
3. `rootComponentId`가 `null`이 아니고 `undefined`도 아닌 표면만 새 Map에 포함하여 반환한다.

---

#### `clearSurfaces(): void`

1. `this.baseProcessor.clearSurfaces()`에 위임.
2. `this.notify()`를 호출하여 변경 사항을 Angular에 알린다.

## 동작 흐름

서버에서 메시지가 수신되면 `processMessages`가 `baseProcessor`에 위임하고 `notify`로 신호를 증가시킨다. 컴포넌트가 `getSurfaces()`를 호출하면 버전 신호 의존성이 등록되어 이후 데이터 변경 시 자동 재평가된다. 사용자 인터랙션 발생 시 `dispatch`가 호출되어 Observable을 통해 이벤트 핸들러(예: 앱 레벨 소켓 통신 레이어)에 전달하고, 핸들러가 `completion.next(responses)`를 호출하면 `dispatch`의 Promise가 해결된다.
