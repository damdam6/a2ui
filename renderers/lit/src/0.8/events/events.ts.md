# renderers/lit/src/0.8/events/events.ts

## 개요

a2ui 렌더러의 커스텀 DOM 이벤트 시스템의 핵심 구현 파일이다. 이벤트 detail 맵 타입(`StateEventDetailMap`), 이벤트 타입-키 일치를 강제하는 유틸리티 타입(`EnforceEventTypeMatch`), 그리고 실제 `CustomEvent`를 래핑하는 `StateEvent` 클래스를 정의한다. 또한 `HTMLElementEventMap`을 전역 확장하여 TypeScript의 DOM 타입 안전성을 확보한다.

## 의존성

### 저장소 내부 모듈
- [`./a2ui.ts`](./a2ui.ts.md) — `A2UIAction` 인터페이스 (`type` import로 참조)
- [`./base.ts`](./base.ts.md) — `BaseEventDetail<EventType>` 인터페이스

### 외부 패키지
없음

## Exports

- `StateEventDetailMap` (타입): 이벤트 타입 키 → detail 인터페이스 매핑 타입
- `StateEvent` (클래스): a2ui 액션을 전달하는 `CustomEvent` 서브클래스

## 상세 명세

### `const eventInit`

**값**: `{ bubbles: true, cancelable: true, composed: true }`

모든 `StateEvent` 인스턴스에 공통 적용되는 `CustomEvent` 초기화 옵션 상수다.
- `bubbles: true` — 이벤트가 DOM 트리를 버블링한다.
- `cancelable: true` — `preventDefault()`로 취소 가능하다.
- `composed: true` — Shadow DOM 경계를 넘어 전파된다.

---

### `type EnforceEventTypeMatch<T extends Record<string, BaseEventDetail<string>>>`

**제네릭**: `T extends Record<string, BaseEventDetail<string>>`

매핑 타입으로, `T`의 각 키 `K`에 대해 해당 값 타입의 `eventType` 필드가 키 `K` 자체와 일치할 때만 그 타입을 유지하고, 일치하지 않으면 `never`로 교체한다. 이를 통해 `StateEventDetailMap`의 키와 각 detail 인터페이스의 `eventType` 문자열이 반드시 동일해야 함을 컴파일 타임에 강제한다.

**조건 분기**:
1. `T[K] extends BaseEventDetail<infer EventType>` 로 `eventType`의 구체 타입을 추론한다.
2. 추론된 `EventType extends K`이면 `T[K]`를 그대로 유지.
3. 아니면 `never`로 대체.

---

### `type StateEventDetailMap`

`EnforceEventTypeMatch`를 적용하여 정의된 구체 매핑 타입:
- 키 `'a2ui.action'` → 값 `A2UI.A2UIAction`

`A2UIAction`의 `eventType`이 `'a2ui.action'`이므로 일치 조건을 만족한다. 향후 이벤트 타입 추가 시 이 맵에 항목을 추가하면 된다.

---

### `class StateEvent<T extends keyof StateEventDetailMap>`

**제네릭**: `T extends keyof StateEventDetailMap` — 현재는 `'a2ui.action'`만 해당

**상속**: `CustomEvent<StateEventDetailMap[T]>` 확장

**정적 필드**:
- `static eventName = 'a2uiaction'` — DOM 이벤트 이름 리터럴. `addEventListener('a2uiaction', ...)` 호출 시 사용하는 문자열.

**생성자**: `constructor(readonly payload: StateEventDetailMap[T])`
- 매개변수 `payload`: `StateEventDetailMap[T]` 타입의 이벤트 상세 객체. `readonly` 클래스 필드로도 저장된다.
- `super(StateEvent.eventName, {detail: payload, ...eventInit})`를 호출하여 `CustomEvent`를 초기화한다. `detail`에 `payload`를 전달하고 `eventInit`의 버블링/취소 가능/composed 옵션을 적용한다.

---

### `declare global { interface HTMLElementEventMap { ... } }`

TypeScript 전역 `HTMLElementEventMap`을 모듈 증강(module augmentation)하여 `'a2uiaction'` 이벤트 이름을 `StateEvent<'a2ui.action'>` 타입과 연결한다. 이를 통해 `element.addEventListener('a2uiaction', handler)` 호출 시 `handler`의 이벤트 매개변수가 `StateEvent<'a2ui.action'>`로 자동 추론된다.

## 동작 흐름

1. 파일 로드 시 `eventInit` 상수가 초기화된다.
2. 소비자가 `new StateEvent({eventType: 'a2ui.action', action: ..., ...})` 형태로 인스턴스를 생성하면, 내부적으로 `'a2uiaction'` 이름의 `CustomEvent`가 생성된다.
3. 생성된 이벤트는 `element.dispatchEvent(event)`로 발행되며, `composed: true` 덕분에 Shadow DOM 경계를 넘어 상위 컴포넌트까지 전파된다.
4. 수신측은 `'a2uiaction'` 이름으로 리스너를 등록하고 `event.detail` 또는 `event.payload`로 액션 정보에 접근한다.
