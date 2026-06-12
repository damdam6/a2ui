# renderers/web_core/src/v0_9/state/data-model.test.ts

## 개요

`DataModel` 클래스의 동작을 Node.js 내장 테스트 러너(`node:test`)로 검증하는 테스트 파일이다. 초기화, 경로 조회, 데이터 변경, 배열 처리, 구독/이벤트 전파, 에러 케이스, 경계 조건 등 전 기능 영역을 커버한다.

## 의존성

### 저장소 내부 모듈

- [`./data-model.ts`](./data-model.ts.md) — `DataModel` 임포트

### 외부 패키지

- `node:assert` — `assert`
- `node:test` — `describe`, `it`, `beforeEach`

## Exports

없음 (테스트 엔트리포인트)

## 테스트 케이스 명세

### 픽스처 (`beforeEach`)

매 테스트 전 다음 구조로 새 `DataModel` 인스턴스를 생성한다:
```
{ user: { name: 'Alice', settings: { theme: 'dark' } }, items: ['a', 'b', 'c'] }
```

---

### 초기화

| 케이스명 | 검증 동작 |
|---|---|
| `initializes with empty data if not provided` | 인수 없이 `new DataModel()` 생성 시 `get('/')` → `{}` |

---

### 기본 조회 (`get`)

| 케이스명 | 검증 동작 |
|---|---|
| `retrieves root data` | `get('/')` → 전체 객체 반환 |
| `retrieves nested path` | `get('/user/name')` → `'Alice'`, `get('/user/settings/theme')` → `'dark'` |
| `retrieves array items` | `get('/items/0')` → `'a'`, `get('/items/1')` → `'b'` |
| `returns undefined for non-existent paths` | 없는 경로 → `undefined` |
| `returns undefined when traversing through undefined/null segments` | `set('/nullable', null)` 후 `/nullable/deep/path` → `undefined` |

---

### 데이터 변경 (`set`)

| 케이스명 | 검증 동작 |
|---|---|
| `sets value at existing path` | `/user/name`을 `'Bob'`으로 교체 |
| `sets value at new path` | `/user/age`에 `30` 신규 생성 |
| `creates intermediate objects` | `/a/b/c`에 `'foo'` 설정 시 `/a/b` 중간 객체 자동 생성 |
| `removes keys when value is undefined` | `undefined` 설정 시 객체 키 삭제 및 `Object.keys`에서 제거 확인 |

---

### 배열/리스트 처리 (Flutter Parity)

| 케이스명 | 검증 동작 |
|---|---|
| `List: set and get` | `/list/0`에 `'hello'` 설정 후 배열임을 확인 |
| `List: append and get` | 인덱스 0, 1 순차 설정 후 길이 2 확인 |
| `List: update existing index` | `/items/1`을 `'updated'`로 교체 |
| `Nested structures are created automatically` | `/a/b/0/c`(배열 내 객체), `/x/y/z`(중첩 객체), `/nestedList/0/0`(중첩 배열)의 자동 생성 확인 |

---

### 구독 (`subscribe`)

| 케이스명 | 검증 동작 |
|---|---|
| `returns a subscription object` | `sub.value`로 현재 값 접근, `set()` 후 `sub.value` 갱신 확인, `unsubscribe()` 후 더 이상 호출되지 않음 |
| `notifies subscribers on exact match` | 정확히 일치하는 경로 변경 시 콜백 호출 |
| `notifies ancestor subscribers (Container Semantics)` | `/user/name` 변경 시 `/user` 구독자에게도 전파 |
| `notifies descendant subscribers` | `/user/settings` 교체 시 `/user/settings/theme` 구독자에게도 전파 |
| `notifies root subscriber` | 하위 경로 변경 시 `'/'` 구독자에게도 전파 |
| `notifies parent when child updates` | 자식 경로 변경 시 부모 구독자 호출 및 값 확인 |
| `stops notifying after dispose` | `dispose()` 후 `set()` 호출해도 콜백 미실행 |
| `supports multiple subscribers to the same path` | 동일 경로에 두 구독자 등록 → 둘 다 각 1회 호출 |
| `allows unsubscribing individual listeners` | 특정 구독 해제 후 해당 콜백만 미호출, 나머지 유지 |
| `handles subscription to non-existent path` | 초기값 `undefined`, `set()` 후 값 갱신 및 콜백 호출 |
| `handles updates to undefined` | 값을 `undefined`로 설정 시 구독자에게 `undefined` 전달 |

---

### 에러 처리

| 케이스명 | 검증 동작 |
|---|---|
| `throws when trying to set nested property through a primitive` | 원시값 아래 경로 설정 시 `/Cannot set path/` 에러 |
| `throws when using non-numeric segment on an array` | 배열에 비숫자 키 설정 시 에러 |
| `throws when using non-numeric segment on an array (intermediate)` | 중간 경로에서 배열에 비숫자 세그먼트 사용 시 에러 (`/Cannot use non-numeric segment 'foo' on an array/`) |
| `throws when path is null or undefined` | `get(null)`, `get(undefined)`, `set(null, ...)`, `set(undefined, ...)` 모두 `/Path cannot be null or undefined/` 에러 |

---

### 경계 케이스

| 케이스명 | 검증 동작 |
|---|---|
| `normalizes trailing slashes` | `/foo/`로 설정 및 구독이 `/foo`와 동일하게 동작 |
| `replaces root object on root update` | `set('/', {...})` 시 `get('')`로 새 루트 확인, `'/'` 구독자 1회 호출 |
| `calculates descendants against root path` | 내부 `isDescendant` 직접 호출: `('/user', '/')` → `true`, `('/', '/')` → `false` |
