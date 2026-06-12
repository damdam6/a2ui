# renderers/lit/src/v0_9/a2ui-controller.ts

## 개요

Lit의 `ReactiveController` 인터페이스를 구현한 `A2uiController<Api>` 클래스를 정의한다. 이 클래스는 특정 A2UI 컴포넌트 API 스키마에 바인딩된 `GenericBinder`를 래핑하고, 바인더의 데이터 변경을 구독하여 Lit 호스트 엘리먼트의 리렌더링을 자동으로 트리거한다. 컴포넌트가 DOM에 연결/분리될 때 구독의 생명주기를 관리하며, `dispose()` 호출로 바인더 자원을 명시적으로 해제할 수 있다.

## 의존성

### 외부 패키지

- `lit` — `ReactiveController`
- `@a2ui/web_core/v0_9` — `GenericBinder`, `ComponentApi`, `ResolveA2uiProps`, `InferredComponentApiSchemaType`

### 저장소 내부 모듈

- [`./a2ui-lit-element.js`](./a2ui-lit-element.ts.md) — `A2uiLitElement`

## Exports

| 이름 | 종류 |
|---|---|
| `A2uiController<Api>` | 클래스 |

## 상세 명세

### 클래스: `A2uiController<Api extends ComponentApi>`

Lit `ReactiveController` 인터페이스를 구현하는 제네릭 클래스다.

#### 필드

| 필드 | 타입 | 접근 | 설명 |
|---|---|---|---|
| `props` | `ResolveA2uiProps<InferredComponentApiSchemaType<Api>>` | `public` | 현재 컴포넌트의 반응형 props. 바인더 업데이트 시마다 교체된다. |
| `binder` | `GenericBinder<InferredComponentApiSchemaType<Api>>` | `private` | 데이터 변경을 감시하는 바인더 인스턴스. |
| `subscription` | `{ unsubscribe: () => void } \| undefined` | `private` | 현재 활성 구독 참조. 구독 전 또는 구독 해제 후에는 `undefined`. |
| `host` | `A2uiLitElement<any>` | `private` (생성자 매개변수로 주입) | 컨트롤러가 연결된 Lit 호스트 엘리먼트. |

#### 생성자: `constructor(host: A2uiLitElement<any>, api: Api)`

1. `new GenericBinder(host.context, api.schema)`로 바인더 인스턴스를 생성한다.
2. `binder.snapshot`으로 초기 `props`를 설정한다.
3. `host.addController(this)`를 호출해 Lit의 컨트롤러 시스템에 자신을 등록한다.
4. `host.isConnected`가 이미 `true`이면 즉시 `hostConnected()`를 호출한다 — 이미 DOM에 연결된 호스트에 컨트롤러를 뒤늦게 추가하는 경우를 처리한다.

#### 메서드: `hostConnected()`

- **시그니처:** `hostConnected(): void`
- Lit 생명주기 콜백 — 호스트가 DOM에 연결될 때 Lit이 호출한다.
- `this.subscription`이 이미 있으면 아무 작업도 하지 않는다(중복 구독 방지).
- `this.subscription`이 없을 때만 `binder.subscribe(callback)`을 호출해 구독을 시작한다.
- 구독 콜백은 `newProps`를 받아 `this.props`를 교체하고, `this.host.requestUpdate()`를 호출해 Lit 리렌더링을 트리거한다.

#### 메서드: `hostDisconnected()`

- **시그니처:** `hostDisconnected(): void`
- Lit 생명주기 콜백 — 호스트가 DOM에서 분리될 때 Lit이 호출한다.
- `this.subscription?.unsubscribe()`를 호출해 데이터 구독을 해제한다.
- `this.subscription`을 `undefined`로 초기화한다.

#### 메서드: `dispose()`

- **시그니처:** `dispose(): void`
- `this.binder.dispose()`를 호출해 `GenericBinder`가 컨텍스트에서 점유하던 자원을 해제한다.
- 컨트롤러가 제거되거나 `context`가 교체될 때 호스트(`A2uiLitElement.willUpdate`)가 직접 호출한다. `hostDisconnected`와 별도로 존재하는 이유는 DOM 분리 없이도 컨텍스트 교체가 발생할 수 있기 때문이다.

## 동작 흐름

생성 시 바인더와 초기 props가 설정된다. DOM 연결 시(`hostConnected`) 구독이 시작되어 데이터 변경마다 props 업데이트 및 호스트 재렌더링이 발생한다. DOM 분리 시(`hostDisconnected`) 구독이 해제되어 메모리 누수를 방지한다. `context` 교체 시에는 호스트의 `willUpdate`가 기존 컨트롤러에 `dispose()`를 호출한 뒤 새 컨트롤러를 생성한다.
