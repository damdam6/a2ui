# renderers/web_core/src/v0_9/state/component-model.ts

## 개요

개별 UI 컴포넌트의 상태를 캡슐화하는 모델 클래스를 정의한다. 각 컴포넌트는 고유 ID, 타입 이름, 그리고 임의의 프로퍼티 딕셔너리를 보유하며, 프로퍼티가 변경될 때 구독자에게 이벤트를 전달한다. dispose 메서드를 통해 내부 이벤트 에미터를 정리하는 간단한 생명주기를 지원한다.

## 의존성

### 저장소 내부 모듈

- [`../common/events.ts`](../common/events.ts.md) — `EventEmitter`, `EventSource` 임포트

### 외부 패키지

없음

## Exports

- `ComponentModel` (클래스)

## 상세 명세

### `ComponentModel` 클래스

컴포넌트 하나의 런타임 상태를 나타내는 클래스.

#### 필드

| 필드 | 종류 | 타입 | 설명 |
|---|---|---|---|
| `id` | `readonly` 공개 | `string` | 컴포넌트의 고유 식별자. 생성자에서 설정되며 변경 불가. |
| `type` | `readonly` 공개 | `string` | 컴포넌트 타입 이름 (예: `'Button'`). 변경 불가. |
| `_properties` | `private` | `Record<string, any>` | 현재 프로퍼티 저장소. setter를 통해서만 교체된다. |
| `_onUpdated` | `private readonly` | `EventEmitter<ComponentModel>` | 프로퍼티 변경 시 이벤트를 발생시키는 내부 에미터. |
| `onUpdated` | `readonly` 공개 | `EventSource<ComponentModel>` | 외부에 노출되는 이벤트 소스. `_onUpdated`의 구독 전용 뷰. |

#### `constructor(id: string, type: string, initialProperties: Record<string, any>)`

- `id`와 `type`을 `readonly` 필드로 설정한다.
- `initialProperties`를 `_properties`에 직접 할당한다. 이 시점에는 이벤트가 발생하지 않는다.

#### `get properties(): Record<string, any>`

내부 `_properties`를 반환한다. 반환된 참조는 직접 수정 가능하지만, 이벤트를 발생시키려면 setter를 사용해야 한다.

#### `set properties(newProperties: Record<string, any>)`

1. `_properties`를 `newProperties`로 교체한다.
2. `_onUpdated.emit(this)`를 호출하여 `ComponentModel` 인스턴스 자신을 인수로 구독자에게 알린다.

#### `dispose(): void`

`_onUpdated.dispose()`를 호출하여 이벤트 에미터의 모든 리스너를 제거하고 리소스를 해제한다.

#### `get componentTree(): any`

컴포넌트의 JSON 직렬화 가능한 표현을 반환한다. `{ id, type, ..._properties }` 형태의 평탄한 객체로, `_properties`의 모든 키·값이 최상위 레벨에 병합된다.

## 동작 흐름

1. 생성자에서 ID·타입·초기 프로퍼티를 받아 내부 상태를 초기화한다.
2. 렌더러 또는 바인더가 `properties` setter를 호출하면 내부 저장소가 교체되고, `onUpdated` 이벤트가 즉시 동기적으로 발생한다.
3. 외부 코드는 `onUpdated`를 구독하여 프로퍼티 변경에 반응할 수 있다.
4. 컴포넌트가 더 이상 필요 없으면 `dispose()`를 호출해 이벤트 에미터를 정리한다.
