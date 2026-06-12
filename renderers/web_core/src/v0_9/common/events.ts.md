# renderers/web_core/src/v0_9/common/events.ts

## 개요

A2UI 내부에서 사용하는 경량 이벤트 발행/구독(pub/sub) 시스템을 정의하는 파일이다. `EventSource<T>` 인터페이스(구독만 허용)와 `EventEmitter<T>` 클래스(발행 포함)를 분리해 외부에는 구독 능력만 노출하는 패턴을 구현한다. `Subscription` 인터페이스로 구독 해제를 표준화하고, 리스너 예외를 격리해 `emit`이 중단되지 않도록 한다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈
없음

## Exports

| 이름 | 종류 | 설명 |
|---|---|---|
| `Subscription` | 인터페이스 | 구독 해제 메서드를 제공하는 표준 정리(cleanup) 인터페이스 |
| `EventListener<T>` | 타입 | 이벤트 데이터를 받는 콜백 함수 타입 |
| `EventSource<T>` | 인터페이스 | 구독만 허용하는 공개 인터페이스 |
| `EventEmitter<T>` | 클래스 | `EventSource<T>`를 구현하며 `emit`과 `dispose`를 추가한 내부 구현체 |

## 상세 명세

### 인터페이스: `Subscription`

| 메서드 | 시그니처 | 설명 |
|---|---|---|
| `unsubscribe` | `(): void` | 이벤트 소스로부터 구독을 해제 |

---

### 타입: `EventListener<T>`

`(data: T) => void | Promise<void>`

이벤트 데이터를 받아 동기 또는 비동기로 처리하는 콜백 함수 타입.

---

### 인터페이스: `EventSource<T>`

모델이 외부에 노출하는 인터페이스로, 구독만 허용한다.

`subscribe(listener: EventListener<T>): Subscription` — 리스너를 등록하고 구독 해제 객체를 반환한다.

---

### `class EventEmitter<T> implements EventSource<T>`

#### 필드

- `private listeners: Set<EventListener<T>>` — 등록된 모든 리스너를 담는 Set. 초기에 빈 Set으로 생성된다.

#### `subscribe(listener: EventListener<T>): Subscription`

- `this.listeners.add(listener)`로 리스너를 Set에 추가한다.
- `{ unsubscribe: () => this.listeners.delete(listener) }` 형태의 `Subscription` 객체를 반환한다. `unsubscribe()` 호출 시 해당 리스너만 Set에서 제거된다.

#### `async emit(data: T): Promise<void>`

- `this.listeners`를 순회(`for...of`)하며 각 `listener(data)`를 `await`로 호출한다.
- 리스너가 예외를 던지면 `try/catch`로 잡아 `console.error('EventEmitter error:', e)`로 로깅한다. 예외가 전파되지 않아 나머지 리스너의 실행이 계속된다.

#### `dispose(): void`

- `this.listeners.clear()`로 모든 리스너를 제거한다.

## 동작 흐름

모델이 `EventEmitter<T>` 인스턴스를 내부적으로 생성하고, `EventSource<T>` 타입으로만 외부에 노출한다. 외부 소비자는 `subscribe()`를 통해 리스너를 등록하고 반환된 `Subscription`으로 해제한다. 모델 내부에서 상태 변경이 발생하면 `emit(data)`를 호출해 모든 리스너에게 비동기적으로 알린다. 컴포넌트 정리 시 `dispose()`를 호출해 메모리 누수를 방지한다.
