# renderers/lit/src/0.8/events/base.ts

## 개요

이벤트 시스템의 모든 이벤트 상세(detail) 인터페이스가 확장해야 하는 기반 타입 `BaseEventDetail`을 정의하는 최소 모듈이다. 제네릭 타입 매개변수로 이벤트 타입 문자열을 강제하여 타입 안전성을 확보한다.

## 의존성

없음 (외부 패키지 및 내부 모듈 import 없음)

## Exports

- `BaseEventDetail<EventType extends string>` (인터페이스)

## 상세 명세

### `interface BaseEventDetail<EventType extends string>`

**제네릭**: `EventType extends string` — 이벤트 타입 문자열을 제네릭 매개변수로 받아 `eventType` 필드의 정확한 리터럴 타입을 보존한다.

**필드**:
| 필드명 | 타입 | 설명 |
|---|---|---|
| `eventType` | `EventType` | 이벤트의 문자열 식별자 (읽기 전용) |

`readonly`로 선언되어 detail 객체 생성 후 `eventType`을 변경할 수 없다.

## 동작 흐름

런타임 코드를 포함하지 않는 순수 타입 파일이다. `a2ui.ts`의 `A2UIAction`과 `events.ts`의 `EnforceEventTypeMatch` 타입 유틸리티가 이 인터페이스를 기반으로 이벤트 타입을 계층적으로 구성한다.
