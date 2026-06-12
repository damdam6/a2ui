# renderers/lit/src/0.8/events/a2ui.ts

## 개요

a2ui 이벤트 시스템에서 사용하는 `A2UIAction` 이벤트 상세(detail) 인터페이스를 정의하는 타입 전용 모듈이다. 사용자 인터랙션으로부터 발생하는 액션 이벤트의 데이터 구조를 명세한다.

## 의존성

### 저장소 내부 모듈
- [`./base.ts`](./base.ts.md) — `BaseEventDetail<EventType>` 제네릭 인터페이스
- 외부 패키지 `@a2ui/web_core/types/types`에서 `Types` 네임스페이스 임포트

### 외부 패키지
- `@a2ui/web_core/types/types` — `Types.Action`, `Types.AnyComponentNode` 타입

## Exports

- `A2UIAction` (인터페이스): a2ui 액션 이벤트의 detail 객체 타입

## 상세 명세

### `type Namespace = 'a2ui'`

파일 내부에서만 사용하는 리터럴 타입 별칭. 이벤트 타입 문자열 접두사를 `'a2ui'`로 고정한다.

### `interface A2UIAction extends BaseEventDetail<\`${Namespace}.action\`>`

**확장 대상**: `BaseEventDetail<'a2ui.action'>` — `eventType: 'a2ui.action'` 필드를 상속한다.

**필드**:
| 필드명 | 타입 | 설명 |
|---|---|---|
| `action` | `Types.Action` | 발생한 액션 정보 (읽기 전용) |
| `dataContextPath` | `string` | 이벤트가 발생한 데이터 컨텍스트 경로 (읽기 전용) |
| `sourceComponentId` | `string` | 이벤트를 발생시킨 컴포넌트의 ID (읽기 전용) |
| `sourceComponent` | `Types.AnyComponentNode \| null` | 이벤트 소스 컴포넌트 노드 객체, 없으면 `null` (읽기 전용) |

모든 필드는 `readonly`로 선언되어 이벤트 detail 객체의 불변성을 보장한다.

## 동작 흐름

이 파일은 타입 정의만 포함하며 런타임 코드를 생성하지 않는다. `events.ts`가 이 인터페이스를 `StateEventDetailMap`의 `'a2ui.action'` 키 값 타입으로 참조한다.
