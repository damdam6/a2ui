# renderers/angular/a2ui_explorer/src/app/action-dispatcher.service.ts

## 개요

`ActionDispatcher`는 Angular 루트 수준에서 제공되는 싱글톤 서비스로, 컴포넌트나 다른 서비스가 `A2uiClientAction`을 발행(dispatch)하고 구독(subscribe)할 수 있도록 RxJS Subject를 기반으로 한 이벤트 버스를 제공한다. v0.9 스펙에서 렌더러와 에이전트 스텁 사이의 액션 흐름을 분리하는 역할을 한다.

## 의존성

### 외부 패키지
- `@angular/core` — `Injectable`
- `rxjs` — `Subject`
- `@a2ui/web_core/v0_9` — `A2uiClientAction`

### 저장소 내부 모듈
없음

## Exports

| 이름 | 종류 |
|---|---|
| `ActionDispatcher` | 클래스 (Angular 서비스) |

## 상세 명세

### 클래스 `ActionDispatcher`

**데코레이터**: `@Injectable({ providedIn: 'root' })`

#### 필드

- `private action$: Subject<A2uiClientAction>`
  - 액션을 발행하는 내부 Subject. 외부에 직접 노출하지 않아 임의의 next 호출을 방지한다.
- `actions: Observable<A2uiClientAction>`
  - `action$.asObservable()`로 생성된 읽기 전용 Observable. 구독자는 이 스트림을 통해 새 액션을 수신한다.

#### 메서드

##### `dispatch(action: A2uiClientAction): void`
- 매개변수: `action` — 발행할 `A2uiClientAction` 객체
- 반환 타입: `void`
- 동작: `action$` Subject의 `next(action)`을 호출해 현재 구독자 모두에게 동기적으로 액션을 전달한다. 에러 처리나 큐잉 없이 단순 전달만 수행한다.

## 동작 흐름

렌더러 설정에서 `actionHandler` 콜백으로 `dispatch`가 등록되어 있다. UI 컴포넌트에서 사용자 이벤트가 발생하면 렌더러가 `dispatch`를 호출하고, `AgentStubV09Service` 등 구독 중인 서비스가 `actions` Observable을 통해 해당 액션을 수신해 처리한다.
