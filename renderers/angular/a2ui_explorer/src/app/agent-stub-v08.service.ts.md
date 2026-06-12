# renderers/angular/a2ui_explorer/src/app/agent-stub-v08.service.ts

## 개요

`AgentStubV08Service`는 v0.8 A2UI 프로토콜을 사용하는 에이전트를 시뮬레이션하는 구체 서비스다. `MessageProcessorV08`과 `ThemeV08`을 주입받아 서버에서 받은 초기화 메시지를 처리하고, 사용자 액션(`update_property`)에 반응해 데이터 모델을 갱신한다. `AgentStubService`를 상속하며 v0.8 전용 테마 기본값과 데이터 모델 조회 방식을 구현한다.

## 의존성

### 외부 패키지
- `@angular/core` — `Injectable`, `signal`, `computed`
- `@a2ui/angular/v0_8` — `MessageProcessor` (별칭 `MessageProcessorV08`), `Theme` (별칭 `ThemeV08`)
- `@a2ui/web_core/v0_9` — `A2uiClientAction`
- `@a2ui/web_core/types/client-event` — `UserAction`

### 저장소 내부 모듈
- [`src/v0_8/types`](../../../../../src/v0_8/types.ts) — `ServerToClientMessage`
- [`./agent-stub.service`](./agent-stub.service.ts.md) — `AgentStubService`

## Exports

| 이름 | 종류 |
|---|---|
| `AgentStubV08Service` | 클래스 (Angular 서비스, `AgentStubService` 상속) |

## 상세 명세

### 로컬 인터페이스

#### `UpdatePropertyContext`
- `path: string` — 업데이트할 데이터 경로 (예: `/name`)
- `value: any` — 새 값
- `surfaceId?: string` — 선택적 Surface ID; 없으면 현재 `surfaceId` 사용

### 클래스 `AgentStubV08Service extends AgentStubService`

**데코레이터**: `@Injectable({ providedIn: 'root' })`

#### 필드

- `override eventsLog: WritableSignal<Array<{ timestamp: Date; action: A2uiClientAction }>>`
  - `signal([])` 로 초기화된 이벤트 로그
- `override surfaceId: WritableSignal<string>`
  - 초기값 `'demo-surface'`, `equal: () => false`로 선언해 동일 값 재설정 시에도 항상 변경 알림을 트리거한다.
- `override currentCreateSurfaceMessage: WritableSignal<ServerToClientMessage[] | null>`
  - 현재 초기화된 메시지 배열 또는 `null`
- `private actionSub?: { unsubscribe: () => void }`
  - 이벤트 구독 참조. `initializeDemo` 재호출 시 기존 구독 해제에 사용된다.
- `override dataModel: Signal<Record<string, unknown>>` (computed)
  - `surfaceId` Signal에 의존하는 computed Signal. `surfaceId`가 비어 있으면 `{}` 반환. 그렇지 않으면 `messageProcessorV08.getSurfaces()`에서 해당 Surface를 조회하고, 존재하면 `messageProcessorV08.getData({ id: 'root' } as any, '/', surfaceId)`로 루트 데이터를 반환, 없으면 `{}` 반환.

#### 생성자

- 매개변수: `messageProcessorV08: MessageProcessorV08`, `themeV08: ThemeV08`
- `super()`를 호출한다.

#### 비공개 메서드

##### `private handleAction(action: UserAction): void`
- v0.8 `UserAction` 이벤트를 처리한다.
- `console.log`로 액션을 출력한 후 50ms 지연(`setTimeout`) 후 실행된다. 이 지연은 렌더러의 동기 처리와 충돌을 방지하기 위한 것이다.
- `action.name === 'update_property'`이고 `context`가 있으면: `context`를 `UpdatePropertyContext`로 캐스팅해 `path`, `value`, `surfaceId`를 추출한다. `messageProcessorV08.processMessages`에 `dataModelUpdate` 메시지를 전달한다. 메시지 구조: `{ dataModelUpdate: { surfaceId, path, contents: [{ key: path.substring(1), valueString: String(value) }] } }`. `key`는 경로에서 앞의 `/`를 제거한 값이다.
- 다른 액션 이름은 처리하지 않는다.

##### `private userActionToClientAction(action: UserAction): A2uiClientAction`
- v0.8 `UserAction`을 `A2uiClientAction`으로 변환한다.
- 변환 규칙: `name`, `surfaceId`, `sourceComponentId`, `timestamp`를 그대로 복사하고, `context`는 `action.context ?? {}`로 null 대비 처리한다.

##### `private getDefault08Theme(): object`
- v0.8 기본 테마 구성 객체를 반환한다.
- 구조는 `{ components: {...}, elements: {...}, markdown: {...} }`이다.
- `components` 키 목록: `AudioPlayer`, `Text`(하위 변형: `all`, `h1`–`h5`, `body`, `caption`), `CheckBox`, `DateTimeInput`, `List`, `Modal`, `MultipleChoice`, `Tabs`(하위: `container`, `element`, `controls.all`, `controls.selected`), `Slider`, `TextField`, `Video`, `Card`, `Row`, `Column`, `Image`(하위 변형 7개), `Divider`, `Icon`, `Button`. 모든 값은 빈 객체 `{}`.
- `elements` 키 목록: `a`, `audio`, `body`, `button`, `h1`–`h5`, `iframe`, `input`, `p`, `pre`, `textarea`, `video`. 모든 값은 빈 객체 `{}`.
- `markdown` 키 목록: `p`, `h1`–`h5`, `ul`, `ol`, `li`, `a`, `strong`, `em`. 모든 값은 빈 배열 `[]`.

#### 공개 오버라이드 메서드

##### `override initializeDemo(initialMessages: ServerToClientMessage[]): void`
- 메시지 배열을 JSON 직렬화/역직렬화로 깊은 복사(`JSON.parse(JSON.stringify(...))`)한다. 이는 원본 메시지 객체 변경을 방지한다.
- `themeV08.update(this.getDefault08Theme())`을 호출해 테마를 기본값으로 초기화한다.
- 복사된 메시지 중 `'surfaceUpdate'` 키를 갖는 메시지를 찾아 `newSurfaceId`를 추출한다. 없으면 기본값 `'demo-surface'`를 사용한다.
- `currentCreateSurfaceMessage.set(clonedMessages)`로 전체 메시지 배열을 저장한다.
- `eventsLog.set([])`로 이벤트 로그를 초기화한다.
- 기존 `actionSub`가 있으면 `unsubscribe()`한다.
- `messageProcessorV08.events`를 구독한다. 각 이벤트에서 `message.userAction`이 있으면 `handleAction`을 호출하고, `userActionToClientAction`으로 변환한 결과를 `eventsLog` 앞에 추가한다.
- `messageProcessorV08.processMessages(clonedMessages)`를 호출해 메시지를 처리한다.
- `surfaceId.set('')`을 호출한 후 `setTimeout(..., 0)`으로 `surfaceId.set(newSurfaceId)`를 비동기 실행한다. 이 패턴은 Angular의 변경 감지를 강제로 트리거하기 위한 것이다.

## 동작 흐름

`initializeDemo` 호출 → 메시지 복사 및 테마 초기화 → 기존 구독 해제 → `messageProcessorV08`에 메시지 처리 위임 → `messageProcessorV08.events` 구독으로 사용자 액션 수신 → `handleAction`에서 50ms 지연 후 `update_property` 액션을 `dataModelUpdate` 메시지로 변환해 `messageProcessorV08`에 재투입 → `surfaceId` Signal의 공/재설정으로 Angular 뷰 갱신 트리거.
