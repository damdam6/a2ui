# renderers/angular/src/v0_8/data/types.ts

## 개요

A2A(Agent-to-Agent) 서버 응답 페이로드의 타입을 정의하는 파일이다. 텍스트 청크와 데이터 메시지 두 가지 페이로드 형태, 그리고 이들의 배열 또는 오류 객체로 구성되는 유니온 타입을 선언한다.

## 의존성

### 저장소 내부 모듈
- [`../types`](../types.ts.md) — `ServerToClientMessage` (타입 전용)

## Exports

| 이름 | 종류 |
|---|---|
| `A2TextPayload` | 인터페이스 |
| `A2DataPayload` | 인터페이스 |
| `A2AServerPayload` | 타입 별칭 |

## 상세 명세

### `A2TextPayload` 인터페이스

서버가 보내는 텍스트 스트리밍 청크를 표현한다.

| 필드 | 타입 | 값 |
|---|---|---|
| `kind` | 리터럴 `'text'` | 페이로드 판별자 |
| `text` | `string` | 텍스트 내용 |

---

### `A2DataPayload` 인터페이스

서버가 보내는 구조화 데이터 메시지를 표현한다.

| 필드 | 타입 | 값 |
|---|---|---|
| `kind` | 리터럴 `'data'` | 페이로드 판별자 |
| `data` | `ServerToClientMessage` | 실제 서버-클라이언트 메시지 객체 |

---

### `A2AServerPayload` 타입 별칭

`Array<A2DataPayload | A2TextPayload> | {error: string}`

서버 응답 전체를 나타내는 최상위 타입. 성공 시 텍스트 또는 데이터 페이로드의 배열이고, 실패 시 `{error: string}` 형태의 오류 객체다. 판별은 `Array.isArray(payload)` 여부로 수행할 수 있다.

## 동작 흐름

이 파일은 타입 선언만 포함하며 런타임 로직이 없다. `A2AServerPayload`를 사용하는 코드는 먼저 배열 여부를 확인하고, 배열인 경우 각 항목의 `kind` 필드로 텍스트 또는 데이터 페이로드를 구분하여 처리한다.
