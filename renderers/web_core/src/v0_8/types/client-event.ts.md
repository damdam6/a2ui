# renderers/web_core/src/v0_8/types/client-event.ts

## 개요

클라이언트(렌더러)가 서버로 전송하는 메시지 및 이벤트의 TypeScript 타입 정의를 모아 놓은 파일이다. UI 클라이언트의 컴포넌트 역량(capabilities) 선언, 사용자 동작(UserAction) 전달, 오류 보고를 위한 인터페이스들을 정의한다. 런타임 로직은 없으며 순수하게 타입 정보만 제공한다.

## 의존성

- 외부 패키지: 없음
- 저장소 내부 모듈: 없음

## Exports

| 이름 | 종류 |
|------|------|
| `ClientCapabilitiesUri` | 타입 별칭 (`string`) |
| `ClientCapabilitiesDynamic` | 인터페이스 |
| `ClientCapabilitiesCatalogUri` | 인터페이스 |
| `ClientCapabilitiesDynamicCatalog` | 인터페이스 |
| `ClientCapabilities` | 타입 (유니언) |
| `ClientToServerMessage` | 인터페이스 |
| `UserAction` | 인터페이스 |
| `ClientError` | 인터페이스 |

## 상세 명세

### `ClientCapabilitiesUri`

- 타입: `string` 별칭
- 역할: 클라이언트가 지원하는 컴포넌트 카탈로그를 가리키는 URI 문자열을 나타낸다.

---

### `ClientCapabilitiesDynamic`

- 필드:
  - `components: {[key: string]: unknown}` — 지원하는 컴포넌트 목록(키-값 맵)
  - `styles: {[key: string]: unknown}` — 지원하는 스타일 목록(키-값 맵)

---

### `ClientCapabilitiesCatalogUri`

- 필드:
  - `catalogUri: ClientCapabilitiesUri` — 컴포넌트 카탈로그 URI

---

### `ClientCapabilitiesDynamicCatalog`

- 필드:
  - `dynamicCatalog: ClientCapabilitiesDynamic` — 동적으로 명세된 역량 정보

---

### `ClientCapabilities`

- 타입: `ClientCapabilitiesCatalogUri | ClientCapabilitiesDynamicCatalog`
- 역할: 클라이언트 역량 표현의 두 가지 방식 중 정확히 하나를 선택하는 유니언 타입. URI 방식과 동적 명세 방식 중 하나만 설정해야 한다.

---

### `ClientToServerMessage`

클라이언트에서 서버로 전송되는 메시지의 최상위 래퍼. 아래 필드 중 정확히 하나만 설정해야 한다는 명시적 계약이 주석으로 기술되어 있다.

- 필드:
  - `userAction?: UserAction` — 사용자가 트리거한 액션
  - `clientUiCapabilities?: ClientCapabilities` — 클라이언트의 UI 역량 선언
  - `error?: ClientError` — 클라이언트 측 오류 보고
  - `request?: unknown` — 데모 콘텐츠용 요청 (자유 형식)

---

### `UserAction`

사용자가 UI에서 발생시킨 동작을 서버에 전달하기 위한 인터페이스.

- 필드:
  - `name: string` — 액션 이름
  - `surfaceId: string` — 이벤트가 발생한 서피스의 ID
  - `sourceComponentId: string` — 이벤트를 트리거한 컴포넌트의 ID
  - `timestamp: string` — ISO 형식의 이벤트 발생 시각
  - `context?: { [k: string]: unknown }` — 컴포넌트의 `action.context`에서 데이터 바인딩이 해소된 이후의 키-값 쌍 (선택 필드)

---

### `ClientError`

렌더링 등 클라이언트 처리 중 발생한 오류를 나타내는 인터페이스.

- 구조: `{ [k: string]: unknown }` — 오류 세부 정보를 임의 키-값으로 자유롭게 담을 수 있는 인덱스 시그니처

## 동작 흐름

이 파일은 타입 선언만 포함하므로 런타임 동작이 없다. 임포트하는 쪽에서 `ClientToServerMessage`를 구성할 때 각 필드를 선택적으로 채우며, 서버 쪽에서는 어떤 필드가 설정되어 있는지를 확인하여 분기 처리한다.
