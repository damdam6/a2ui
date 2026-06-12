# renderers/angular/a2ui_explorer/src/app/agent-stub.service.ts

## 개요

`AgentStubService`는 A2UI 탐색기에서 실제 에이전트 백엔드를 대체하는 스텁 서비스들의 추상 기반 클래스다. v0.8과 v0.9 두 가지 프로토콜 버전을 지원하는 구체 서비스(`AgentStubV08Service`, `AgentStubV09Service`)가 이 클래스를 상속하며, Angular DI 토큰으로 사용되어 버전에 따라 적절한 구현체로 교체된다.

## 의존성

### 외부 패키지
- `@angular/core` — `WritableSignal`, `Signal`
- `@a2ui/web_core/v0_9` — `A2uiClientAction`, `A2uiMessage`, `CreateSurfaceMessage`

### 저장소 내부 모듈
- [`src/v0_8/types`](../../../../../src/v0_8/types.ts) — `ServerToClientMessage`

## Exports

| 이름 | 종류 |
|---|---|
| `AgentStubService` | 추상 클래스 |

## 상세 명세

### 추상 클래스 `AgentStubService`

데코레이터 없이 일반 추상 클래스로 선언된다. Angular DI에서는 `{ provide: AgentStubService, useFactory: ... }` 패턴으로 버전별 구체 클래스가 주입된다.

#### 추상 필드

- `abstract eventsLog: WritableSignal<Array<{ timestamp: Date; action: A2uiClientAction }>>`
  - 사용자 액션 이벤트의 시간 기록 로그. 가장 최근 항목이 배열 앞에 위치한다.
- `abstract dataModel: Signal<Record<string, unknown>>`
  - 현재 활성 Surface의 데이터 모델 상태. 읽기 전용 Signal이다.
- `abstract surfaceId: Signal<string>`
  - 현재 렌더링 중인 Surface의 ID. 빈 문자열이면 Surface가 없음을 의미한다.
- `abstract currentCreateSurfaceMessage: Signal<CreateSurfaceMessage | ServerToClientMessage[] | null>`
  - 현재 예제의 Surface 생성 메시지. v0.9에서는 `CreateSurfaceMessage`, v0.8에서는 `ServerToClientMessage[]`이다.

#### 추상 메서드

##### `abstract initializeDemo(initialMessages: A2uiMessage[] | ServerToClientMessage[]): void`
- 매개변수: `initialMessages` — v0.9(`A2uiMessage[]`) 또는 v0.8(`ServerToClientMessage[]`) 초기화 메시지 배열
- 반환 타입: `void`
- 동작: 구체 구현에서 초기화 메시지를 처리해 Surface를 생성하고, 기존 구독을 정리한 후 새 액션 구독을 설정한다.

## 동작 흐름

이 파일은 인터페이스 계약만 정의하며 런타임 로직을 포함하지 않는다. `DemoComponent`의 providers 배열에서 `A2UI_VERSION` 토큰 값에 따라 `AgentStubV09Service` 또는 `AgentStubV08Service` 중 하나를 `AgentStubService` 토큰으로 제공한다.
