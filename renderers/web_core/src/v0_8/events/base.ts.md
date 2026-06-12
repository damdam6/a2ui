# renderers/web_core/src/v0_8/events/base.ts

## 개요

A2UI 이벤트 시스템의 최상위 기반 인터페이스를 정의하는 소형 파일이다. 모든 A2UI 커스텀 이벤트의 `detail` 페이로드가 반드시 `eventType` 프로퍼티를 포함하도록 강제하는 제네릭 인터페이스 `BaseEventDetail`을 export한다.

## 의존성

### 외부 패키지
- (없음)

### 저장소 내부 모듈
- (없음)

## Exports

| 이름 | 종류 |
|------|------|
| `BaseEventDetail` | 인터페이스 (제네릭) |

## 상세 명세

### 인터페이스 `BaseEventDetail<EventType extends string>`

- 타입 매개변수 `EventType`은 `string`을 상한으로 갖는 제네릭으로, 이벤트 이름 리터럴 타입을 표현한다.
- `readonly eventType: EventType` — 이벤트 이름과 동일한 값을 `detail` 페이로드에 포함시켜, `detail` 객체만으로도 이벤트 종류를 식별할 수 있게 한다.

## 동작 흐름

이 파일은 타입 선언만 포함하며 런타임 코드가 없다. 다른 이벤트 파일에서 `extends BaseEventDetail<'이벤트-이름'>` 형태로 상속해 사용한다.
