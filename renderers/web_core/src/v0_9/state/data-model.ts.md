# renderers/web_core/src/v0_9/state/data-model.ts

## 개요

클라이언트 사이드 상태를 담는 관찰 가능한(reactive) 데이터 스토어를 구현한다. JSON Pointer 스타일의 경로(`/user/name`, `/items/0` 등)를 사용한 읽기·쓰기를 지원하며, Preact Signals를 내부 반응성 기반으로 사용하여 경로 단위 구독과 조상/자손 경로까지의 전파를 제공한다. Flutter 구현과의 동작 일치를 목표로 하며, 배열과 객체를 자동으로 중간 생성한다.

## 의존성

### 저장소 내부 모듈

- [`../common/events.ts`](../common/events.ts.md) — `Subscription` 타입 임포트
- [`../errors.ts`](../errors.ts.md) — `A2uiDataError` 임포트

### 외부 패키지

- `@preact/signals-core` — `signal`, `Signal`, `batch`, `effect`

## Exports

- `DataSubscription<T>` (인터페이스)
- `DataModel` (클래스)

## 상세 명세

### `DataSubscription<T>` 인터페이스

`BaseSubscription`(`Subscription`)을 확장한다.

| 필드 | 타입 | 설명 |
|---|---|---|
| `value` | `T \| undefined` (readonly) | 구독 중인 경로의 현재 값. 변경 시 자동으로 갱신된다. |

`unsubscribe()` 메서드는 `BaseSubscription`에서 상속된다.

---

### `isNumeric(value: string): boolean` (비공개 헬퍼 함수)

정규식 `/^\d+$/`로 문자열이 순수 숫자인지 확인한다. 경로 세그먼트가 배열 인덱스인지 판별하는 데 사용된다.

---

### `DataModel` 클래스

#### 필드

| 필드 | 종류 | 타입 | 설명 |
|---|---|---|---|
| `data` | `private` | `Record<string, unknown>` | 실제 데이터가 저장되는 루트 객체. |
| `signals` | `private readonly` | `Map<string, Signal<any>>` | 경로 → Preact Signal 매핑 테이블. |
| `subscriptions` | `private readonly` | `Set<() => void>` | `effect()` 반환 dispose 함수 집합. `dispose()` 시 전부 해제한다. |

#### `constructor(initialData: Record<string, unknown> = {})`

`initialData`를 `data`에 직접 할당한다. 기본값은 빈 객체 `{}`.

---

#### `getSignal<T>(path: string): Signal<T | undefined>`

1. `normalizePath(path)`로 경로를 정규화한다.
2. `signals` 맵에 해당 경로가 없으면 `signal(this.get(normalizedPath))`로 새 Signal을 생성하고 저장한다.
3. 해당 Signal을 반환한다.

Signal은 항상 경로의 현재 값으로 초기화된다. 이후 `set()`이 호출되면 `notifySignals()`를 통해 자동 업데이트된다.

---

#### `set(path: string, value: any): this`

JSON Pointer 경로에 값을 쓴다.

1. `path`가 `null` 또는 `undefined`이면 `A2uiDataError('Path cannot be null or undefined.')` 던짐.
2. `path`가 `'/'` 또는 `''`이면 `data` 전체를 `value`로 교체하고 `notifyAllSignals()` 호출 후 반환.
3. `parsePath(path)`로 세그먼트 배열을 만들고 마지막 세그먼트를 `pop()`으로 분리.
4. 중간 세그먼트를 순서대로 탐색하며 컨테이너를 생성한다:
   - 현재 컨테이너가 배열인데 세그먼트가 숫자가 아니면 `A2uiDataError('Cannot use non-numeric segment ...')` 던짐.
   - 현재 세그먼트 값이 객체가 아닌 원시값(non-null/undefined)이면 `A2uiDataError('Cannot set path ...: segment is a primitive value.')` 던짐.
   - 값이 `undefined` 또는 `null`이면, 다음 세그먼트(`i+1`번째 또는 `lastSegment`)가 숫자인지 보고 배열(`[]`) 또는 객체(`{}`)를 자동 생성.
5. 마지막 세그먼트에 도달한 컨테이너에서:
   - 컨테이너가 배열인데 `lastSegment`가 숫자가 아니면 에러.
   - `value === undefined`이면: 배열이면 해당 인덱스에 `undefined` 설정(sparse), 객체이면 `delete current[lastSegment]`.
   - 그 외에는 `current[lastSegment] = value`.
6. `notifySignals(path)` 호출 후 `this` 반환.

---

#### `get(path: string): any`

JSON Pointer 경로에서 값을 읽는다.

1. `path`가 `null` 또는 `undefined`이면 `A2uiDataError` 던짐.
2. `path`가 `'/'` 또는 `''`이면 `data` 전체 반환.
3. `parsePath(path)`로 세그먼트 배열 생성.
4. 세그먼트를 순서대로 탐색하며 `current = current[segment]`. 중간에 `undefined` 또는 `null`이면 즉시 `undefined` 반환.

---

#### `subscribe<T>(path: string, onChange: (value: T | undefined) => void): DataSubscription<T>`

특정 경로 변경을 구독한다.

1. `getSignal<T>(path)`로 Signal을 얻는다.
2. `isSync = true`로 설정한 뒤 `effect()`를 호출한다. effect 내부에서 `sig.value`를 읽어 `currentValue`를 갱신하고, `isSync`가 `false`일 때만 `onChange(val)`를 호출한다.
3. `isSync = false`로 설정. (초기 effect 실행은 동기이므로 최초 호출을 무시하는 패턴)
4. `dispose` 함수를 `subscriptions` 집합에 추가.
5. `{ get value() { return currentValue; }, unsubscribe() { dispose(); subscriptions.delete(dispose); } }` 형태의 `DataSubscription` 반환.

---

#### `dispose(): void`

1. `subscriptions` 집합의 모든 dispose 함수를 호출.
2. `subscriptions`와 `signals` 맵을 비운다.

---

#### `normalizePath(path: string): string` (private)

- 경로가 길이 1 초과이고 `'/'`로 끝나면 마지막 슬래시를 제거한다.
- 빈 문자열이면 `'/'`로 정규화한다.
- 그 외에는 그대로 반환.

---

#### `parsePath(path: string): string[]` (private)

`path.split('/').filter(p => p.length > 0)`으로 세그먼트 배열을 반환. 선행 슬래시(`/`)로 생기는 빈 문자열은 필터링된다.

---

#### `notifySignals(path: string): void` (private)

경로 변경을 관련된 모든 Signal에 전파한다.

1. `normalizePath(path)`로 정규화.
2. Preact `batch()` 내에서:
   - `updateSignal(normalizedPath)` 호출 (정확히 변경된 경로).
   - `parentPath`를 루트(`'/'`)에 도달할 때까지 `lastIndexOf('/')` 로 잘라가며 각 조상 경로에 `updateSignal()` 호출.
   - 등록된 모든 Signal 경로를 순회하며 `isDescendant(subPath, normalizedPath)`가 참인 경로에 `updateSignal()` 호출.

---

#### `updateSignal(path: string): void` (private)

`signals` 맵에 해당 경로의 Signal이 있으면 현재 값을 `get(path)`로 읽어 업데이트한다.
- 배열이면 `[...val]` (얕은 복사)
- 객체이면 `{...val}` (얕은 복사)
- 원시값이면 그대로 할당

얕은 복사를 통해 Signal 값이 이전 참조와 다른 새 참조가 되어, Signal의 equality check를 통과하고 effect가 재실행된다.

---

#### `notifyAllSignals(): void` (private)

`batch()` 내에서 등록된 모든 Signal 경로에 `updateSignal()`을 호출한다. 루트(`/`, `''`) 교체 시 사용된다.

---

#### `isDescendant(childPath: string, parentPath: string): boolean` (private)

- `parentPath`가 `'/'` 또는 `''`이면: `childPath !== '/'`인 경우 `true`.
- 그 외: `childPath.startsWith(parentPath + '/')`.

## 동작 흐름

1. 인스턴스 생성 시 초기 데이터가 `data`에 저장된다.
2. `set(path, value)` 호출 → 데이터 트리 변이 → `notifySignals(path)` 호출 → 해당 경로, 모든 조상 경로, 모든 자손 경로의 Signal이 `batch()` 내에서 일괄 업데이트된다.
3. `subscribe(path, cb)` 호출 시 Preact `effect()`로 Signal을 감시하는 리액티브 구독이 생성된다. Signal 값이 바뀌면 `onChange` 콜백이 비동기적으로 실행된다.
4. `dispose()` 호출 시 모든 effect dispose 함수가 실행되어 구독이 완전히 해제된다.
