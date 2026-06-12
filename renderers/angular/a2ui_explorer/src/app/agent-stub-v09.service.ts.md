# renderers/angular/a2ui_explorer/src/app/agent-stub-v09.service.ts

## 개요

`AgentStubV09Service`는 v0.9 A2UI 프로토콜을 사용하는 에이전트를 시뮬레이션하는 구체 서비스다. `A2uiRendererService`를 통해 Surface를 생성하고, `ActionDispatcher`를 구독해 `update_property` 및 `submit_form` 액션에 반응하여 데이터 모델을 갱신한다. `AgentStubService`를 상속하며 v0.9 메시지 구조를 사용한다.

## 의존성

### 외부 패키지
- `@angular/core` — `Injectable`, `signal`
- `@a2ui/angular/v0_9` — `A2uiRendererService`
- `@a2ui/web_core/v0_9` — `A2uiClientAction`, `A2uiMessage`, `CreateSurfaceMessage`

### 저장소 내부 모듈
- [`./action-dispatcher.service`](./action-dispatcher.service.ts.md) — `ActionDispatcher`
- [`./agent-stub.service`](./agent-stub.service.ts.md) — `AgentStubService`

## Exports

| 이름 | 종류 |
|---|---|
| `AgentStubV09Service` | 클래스 (Angular 서비스, `AgentStubService` 상속) |

## 상세 명세

### 로컬 인터페이스

#### `UpdatePropertyContext`
- `path: string` — 업데이트할 데이터 경로 (예: `/name`)
- `value: any` — 새 값
- `surfaceId?: string` — 선택적 Surface ID; 없으면 액션의 `surfaceId` 사용

#### `SubmitFormContext`
- 인덱스 시그니처 `[key: string]: any` — 폼 필드 데이터
- `name?: string` — 선택적 이름 필드

### 클래스 `AgentStubV09Service extends AgentStubService`

**데코레이터**: `@Injectable({ providedIn: 'root' })`

#### 필드

- `override dataModel: WritableSignal<Record<string, unknown>>`
  - `signal({})` 로 초기화. Surface의 데이터 모델을 직접 구독하여 동기화된다.
- `override surfaceId: WritableSignal<string>`
  - 초기값 `'demo-surface'`, `equal: () => false`로 선언해 항상 변경 알림을 트리거한다.
- `override eventsLog: WritableSignal<Array<{ timestamp: Date; action: A2uiClientAction }>>`
  - `signal([])` 로 초기화된 이벤트 로그
- `override currentCreateSurfaceMessage: WritableSignal<CreateSurfaceMessage | null>`
  - 현재 초기화된 `createSurface` 메시지 또는 `null`
- `private actionSub?: { unsubscribe: () => void }`
  - `ActionDispatcher.actions` 구독 참조
- `private dataModelSub?: { unsubscribe: () => void }`
  - Surface 데이터 모델 변경 구독 참조

#### 생성자

- 매개변수: `rendererService: A2uiRendererService`, `actionDispatcher: ActionDispatcher`
- `super()`를 호출한다.

#### 비공개 메서드

##### `private handleAction(action: A2uiClientAction): void`
- v0.9 `A2uiClientAction`을 처리한다.
- `console.log`로 액션을 출력한 후 50ms 지연(`setTimeout`) 후 실행된다.
- **`update_property` 분기**: `context`를 `UpdatePropertyContext`로 캐스팅한다. `path`, `value`, `surfaceId`를 추출한다. `console.log`로 상세 정보를 출력한다. `rendererService.processMessages`에 `version: 'v0.9'`, `updateDataModel: { surfaceId: surfaceId || action.surfaceId, path, value }` 구조의 메시지 1개를 전달한다. `surfaceId` 우선순위: context의 값 → action의 값.
- **`submit_form` 분기**: `context`를 `SubmitFormContext`로 캐스팅한다. `formData.name || 'Anonymous'`로 이름을 추출한다. `rendererService.processMessages`에 메시지 2개를 전달한다: (1) `path: '/form/submitted'`, `value: true`; (2) `path: '/form/responseMessage'`, `value: 'Hello, ${nameValue}! Your form has been processed.'`. 두 메시지 모두 `version: 'v0.9'`, `updateDataModel`, `surfaceId: action.surfaceId` 구조다.
- 다른 액션 이름은 처리하지 않는다.

#### 공개 오버라이드 메서드

##### `override initializeDemo(initialMessages: A2uiMessage[]): void`
- 메시지 배열을 JSON 직렬화/역직렬화로 깊은 복사한다.
- **기존 Surface 삭제 처리**: `rendererService.surfaceGroup`이 존재하면, 복사된 메시지 중 `'createSurface'` 키를 갖는 메시지들을 순회한다. 각 메시지의 `createSurface.surfaceId`로 `surfaceGroup.getSurface`를 호출해 이미 존재하면 `{ version: 'v0.9', deleteSurface: { surfaceId } }` 메시지를 `processMessages`에 먼저 전달한다.
- 복사된 메시지 중 첫 번째 `createSurface` 키를 갖는 메시지를 `CreateSurfaceMessage`로 타입 가드하여 `createMsg`로 추출한다. `newSurfaceId`는 `createMsg?.createSurface.surfaceId ?? 'demo-surface'`이다.
- `currentCreateSurfaceMessage.set(createMsg || null)`을 호출한다.
- `eventsLog.set([])`로 이벤트 로그를 초기화한다.
- 기존 `actionSub`가 있으면 `unsubscribe()`한다. `actionDispatcher.actions`를 구독하여 각 액션에 대해 `handleAction`을 호출하고 `eventsLog` 앞에 추가한다.
- `rendererService.processMessages(clonedMessages)`를 호출해 메시지를 처리한다.
- 기존 `dataModelSub`가 있으면 `unsubscribe()`한다. `rendererService.surfaceGroup?.getSurface(newSurfaceId)`로 Surface를 조회한다. Surface와 `dataModel`이 존재하면 `surface.dataModel.subscribe('/', data => this.dataModel.set(...))` 구독을 설정하고 `dataModel.set(surface.dataModel.get('/'))`으로 초기 상태를 동기화한다. Surface가 없으면 `dataModel.set({})`을 호출한다.
- `surfaceId.set('')` 후 `setTimeout(..., 0)`으로 `surfaceId.set(newSurfaceId)`를 비동기 실행한다.

## 동작 흐름

`initializeDemo` 호출 → 기존 Surface 존재 시 삭제 메시지 선전송 → 전체 메시지 처리(Surface 생성 포함) → `ActionDispatcher` 구독으로 사용자 액션 수신 → `handleAction`에서 50ms 지연 후 `update_property`/`submit_form`에 따라 `rendererService`에 데이터 모델 업데이트 메시지 투입 → Surface 데이터 모델 변경 구독으로 `dataModel` Signal 동기화 → `surfaceId` 공/재설정으로 Angular 뷰 갱신 트리거.
