# renderers/web_core/src/v0_8/events/validation-event.ts

## 개요

입력 컴포넌트의 유효성 검사 상태가 변경될 때 발생하는 커스텀 DOM 이벤트를 정의한다. `ValidationEventDetail` 인터페이스로 페이로드 구조를 명세하고, `A2UIValidationEvent` 클래스로 실제 `CustomEvent`를 구성한다. 또한 `HTMLElementEventMap`을 전역 보강(global augmentation)해 TypeScript 타입 시스템에서 `addEventListener('a2ui-validation-input', ...)` 호출 시 올바른 타입 추론이 가능하도록 한다.

## 의존성

### 외부 패키지
- (없음)

### 저장소 내부 모듈
- [`./base.js`](./base.ts.md) — `BaseEventDetail`

## Exports

| 이름 | 종류 |
|------|------|
| `ValidationEventDetail` | 인터페이스 |
| `A2UIValidationEvent` | 클래스 |

## 상세 명세

### 인터페이스 `ValidationEventDetail extends BaseEventDetail<'a2ui-validation-input'>`

`CustomEvent<ValidationEventDetail>`의 `detail` 페이로드 타입.

| 프로퍼티 | 타입 | 설명 |
|----------|------|------|
| `eventType` | `'a2ui-validation-input'` | (`BaseEventDetail`에서 상속) 이벤트 이름과 동일한 리터럴 타입 |
| `componentId` | `string` | 검증 대상 컴포넌트의 ID |
| `value` | `string` | 입력 컴포넌트의 현재 값 |
| `valid` | `boolean` | 현재 값이 유효성 제약 조건을 충족하는지 여부 |

모든 프로퍼티는 `readonly`이다.

---

### 클래스 `A2UIValidationEvent extends CustomEvent<ValidationEventDetail>`

#### 정적 상수

- `static readonly EVENT_NAME = 'a2ui-validation-input'` — 이벤트 이름 상수. 이벤트 리스너 등록 및 `CustomEvent` 생성자의 첫 번째 인자로 사용된다.

#### 생성자 `constructor(detail: Omit<ValidationEventDetail, 'eventType'>, eventInitDict?: EventInit)`

- `detail` 인자는 `eventType`을 제외한 나머지 필드(`componentId`, `value`, `valid`)를 받는다.
- `super` 호출:
  - 첫 번째 인자: `A2UIValidationEvent.EVENT_NAME`
  - 두 번째 인자: `EventInit` 옵션 객체로, 기본값으로 `bubbles: true`, `composed: true`를 설정하고, `eventInitDict`로 재정의 가능하다.
  - `detail` 필드: 전달받은 `detail`에 `eventType: A2UIValidationEvent.EVENT_NAME`을 추가해 완전한 `ValidationEventDetail`을 구성한다.
- `bubbles: true`로 DOM 버블링이 발생하며, `composed: true`로 Shadow DOM 경계를 넘어 전파된다.

---

### 전역 보강 `declare global`

```ts
interface HTMLElementEventMap {
  'a2ui-validation-input': A2UIValidationEvent;
}
```

TypeScript의 `HTMLElementEventMap`에 `'a2ui-validation-input'` 키를 추가해, `element.addEventListener('a2ui-validation-input', handler)`에서 `handler`의 이벤트 타입이 `A2UIValidationEvent`로 추론되게 한다.

## 동작 흐름

입력 컴포넌트가 유효성 상태 변화를 감지하면 `new A2UIValidationEvent({ componentId, value, valid })`로 인스턴스를 생성하고 `element.dispatchEvent(...)`로 발송한다. 이벤트는 DOM 트리를 버블링하며 Shadow DOM 경계도 통과하므로, 렌더러 호스트나 상위 레이아웃 컴포넌트가 중앙에서 수신할 수 있다.
