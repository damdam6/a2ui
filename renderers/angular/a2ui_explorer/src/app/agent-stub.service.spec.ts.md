# renderers/angular/a2ui_explorer/src/app/agent-stub.service.spec.ts

## 개요

`AgentStubService`의 통합 테스트 파일이다. `AgentStubV09Service`를 `AgentStubService` 토큰으로 제공한 후 `initializeDemo`의 핵심 동작, 특히 Surface 중복 생성 방지를 위한 `deleteSurface` 메시지 선전송 로직을 검증한다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `TestBed`
- `rxjs` — `Subject`
- `@a2ui/angular/v0_9` — `A2uiRendererService`
- `@a2ui/web_core/v0_9` — `A2uiMessage`

### 저장소 내부 모듈
- [`./agent-stub.service`](./agent-stub.service.ts.md) — `AgentStubService`
- [`./agent-stub-v09.service`](./agent-stub-v09.service.ts.md) — `AgentStubV09Service`
- [`./action-dispatcher.service`](./action-dispatcher.service.ts.md) — `ActionDispatcher`
- [`./types`](./types.ts) — `A2UI_VERSION`, `Version`

## 테스트 케이스 명세

### describe: `'AgentStubService'`

#### 픽스처 및 모킹

**beforeEach 설정:**
- `mockSurfaceGroup`: `getSurface` spy를 가진 객체. `A2uiRendererService.surfaceGroup` 게터가 이 객체를 반환한다.
- `mockRendererService`: `processMessages` spy를 가진 객체. `surfaceGroup` 게터를 통해 `mockSurfaceGroup`에 접근한다.
- `mockActionDispatcher`: `actions` 필드로 `new Subject()`를 가진 객체.
- `TestBed.configureTestingModule` providers:
  - `AgentStubService` → `AgentStubV09Service`로 교체
  - `A2uiRendererService` → `mockRendererService`로 교체
  - `ActionDispatcher` → `mockActionDispatcher`로 교체
  - `A2UI_VERSION` → `Version.V0_9`로 교체
- `service = TestBed.inject(AgentStubService)`로 서비스 인스턴스를 획득한다.

#### 테스트 케이스 1: `'should be created'`
- **검증 동작**: `TestBed.inject(AgentStubService)`로 주입한 서비스 인스턴스가 truthy임을 확인한다.
- **픽스처**: beforeEach 설정 그대로 사용.

#### describe: `'initializeDemo'`

공통 픽스처: `surfaceId: 'test-surface'`인 `createSurface` 메시지 1개를 담은 `messages` 배열.

##### 테스트 케이스 2: `'should send deleteSurface before createSurface if surface already exists'`
- **검증 동작**: 동일 Surface를 두 번 초기화할 때, 두 번째 호출에서 `processMessages`가 정확히 2번 호출되는지 확인한다. 첫 번째 호출은 `deleteSurface` 메시지(`{ version: 'v0.9', deleteSurface: { surfaceId } }`), 두 번째 호출은 원본 `messages` 배열 그대로임을 검증한다.
- **시나리오**:
  1. `mockSurfaceGroup.getSurface`가 `undefined`를 반환하도록 설정 → `service.initializeDemo(messages)` 호출 → `processMessages`가 1번만, `messages`를 인자로 호출됐는지 확인 → spy 초기화.
  2. `mockSurfaceGroup.getSurface`가 `{ id: surfaceId }`를 반환하도록 설정 → `service.initializeDemo(messages)` 재호출 → `processMessages.calls.allArgs()`가 `[[deleteMessages], [messages]]`와 정확히 일치하는지 확인.
- **모킹**: `getSurface` spy의 returnValue를 첫/두 번째 호출마다 다르게 설정.

##### 테스트 케이스 3: `'should NOT send deleteSurface if surface does not exist'`
- **검증 동작**: Surface가 존재하지 않을 때 `initializeDemo`를 호출하면 `processMessages`가 1번만 호출되고, 인자는 원본 `messages`임을 확인한다. `deleteSurface` 메시지가 전송되지 않음을 보장한다.
- **시나리오**: `mockSurfaceGroup.getSurface`가 항상 `undefined`를 반환하도록 설정 후 `service.initializeDemo(messages)` 호출.
- **모킹**: `getSurface` spy의 returnValue를 `undefined`로 설정.
