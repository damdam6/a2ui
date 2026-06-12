# agent_sdks/python/a2ui_core/src/a2ui/core/schema/client_to_server.py

## 개요

A2UI 클라이언트가 서버로 전송하는 모든 메시지 유형의 Pydantic 모델을 정의한다. 자동 생성된 파일로, 액션 이벤트 메시지, 오류 메시지, 현재 데이터 모델 스냅샷, 이들을 감싸는 목록 래퍼를 포함한다. 현재 명세 버전(`SPEC_VERSION`)이 모든 메시지의 `version` 필드에 리터럴 타입으로 고정된다.

## 의존성

### 외부 패키지
- `typing` — `Any`, `Dict`, `List`, `Literal`, `Optional`, `Union`
- `pydantic` — `BaseModel`, `Field`, `ConfigDict`

### 저장소 내부 모듈
- [`./common_types.py`](common_types.py.md) — `StrictBaseModel`
- [`./constants.py`](constants.py.md) — `SPEC_VERSION`

## Exports

| 이름 | 종류 |
|---|---|
| `A2uiClientAction` | 클래스 (Pydantic 모델) |
| `A2uiValidationError` | 클래스 (Pydantic 모델) |
| `A2uiGenericError` | 클래스 (Pydantic 모델) |
| `A2uiClientError` | 타입 별칭 (Union) |
| `A2uiClientActionMessage` | 클래스 (Pydantic 모델) |
| `A2uiClientErrorMessage` | 클래스 (Pydantic 모델) |
| `A2uiClientMessage` | 타입 별칭 (Union) |
| `A2uiClientDataModel` | 클래스 (Pydantic 모델) |
| `A2uiClientMessageList` | 타입 별칭 (List) |
| `A2uiClientMessageListWrapper` | 클래스 (Pydantic 모델) |

## 상세 명세

### `A2uiClientAction(StrictBaseModel)`

클라이언트에서 발생한 사용자 액션 이벤트를 표현한다.

| 필드명 | 타입 | alias | 필수 | 설명 |
|---|---|---|---|---|
| `name` | `str` | — | O | 액션 이름. 컴포넌트의 `action.event.name` 값. |
| `surface_id` | `str` | `"surfaceId"` | O | 이벤트가 발생한 서피스의 ID. |
| `source_component_id` | `str` | `"sourceComponentId"` | O | 이벤트를 발생시킨 컴포넌트의 ID. |
| `timestamp` | `str` | — | O | 이벤트 발생 시각. ISO 8601 형식 문자열. |
| `context` | `Dict[str, Any]` | — | O | 액션 컨텍스트. `action.event.context`에서 데이터 바인딩을 모두 해소한 키-값 쌍. |

---

### `A2uiValidationError(StrictBaseModel)`

카탈로그 스키마 검증 실패를 표현하는 오류 모델.

| 필드명 | 타입 | alias | 필수 | 설명 |
|---|---|---|---|---|
| `code` | `Literal["VALIDATION_FAILED"]` | — | 기본값 `"VALIDATION_FAILED"` | 오류 코드. 항상 `"VALIDATION_FAILED"`. |
| `surface_id` | `str` | `"surfaceId"` | O | 오류가 발생한 서피스 ID. |
| `path` | `str` | — | O | 검증 실패 필드를 가리키는 JSON 포인터 (예: `"/components/0/text"`). |
| `message` | `str` | — | O | 검증 실패 이유를 1~2 문장으로 설명. |

---

### `A2uiGenericError(BaseModel)`

일반 오류를 표현하는 모델. `StrictBaseModel` 대신 `BaseModel`을 직접 상속하며 `ConfigDict(populate_by_name=True)`를 적용한다. extra 필드 금지가 없으므로 코드 등 임의 타입을 허용한다.

| 필드명 | 타입 | alias | 필수 | 설명 |
|---|---|---|---|---|
| `code` | `Any` | — | O | 임의 타입의 오류 코드. |
| `message` | `str` | — | O | 오류 이유를 1~2 문장으로 설명. |
| `surface_id` | `str` | `"surfaceId"` | O | 오류가 발생한 서피스 ID. |

---

### `A2uiClientError`

`Union[A2uiValidationError, A2uiGenericError]` — 클라이언트 오류의 두 가지 형태를 통합한 타입 별칭.

---

### `A2uiClientActionMessage(StrictBaseModel)`

액션 이벤트를 담는 메시지 봉투.

| 필드명 | 타입 | 필수 | 설명 |
|---|---|---|---|
| `version` | `Literal[SPEC_VERSION]` | 기본값 `SPEC_VERSION` | 명세 버전. 현재 `"v0.9"`. |
| `action` | `A2uiClientAction` | O | 액션 페이로드. |

---

### `A2uiClientErrorMessage(StrictBaseModel)`

오류를 담는 메시지 봉투.

| 필드명 | 타입 | 필수 | 설명 |
|---|---|---|---|
| `version` | `Literal[SPEC_VERSION]` | 기본값 `SPEC_VERSION` | 명세 버전. |
| `error` | `A2uiClientError` | O | 오류 페이로드. |

---

### `A2uiClientMessage`

`Union[A2uiClientActionMessage, A2uiClientErrorMessage]` — 클라이언트→서버 메시지의 두 가지 형태를 통합한 타입 별칭.

---

### `A2uiClientDataModel(StrictBaseModel)`

클라이언트의 현재 데이터 모델 스냅샷.

| 필드명 | 타입 | 필수 | 설명 |
|---|---|---|---|
| `version` | `Literal[SPEC_VERSION]` | 기본값 `SPEC_VERSION` | 명세 버전. |
| `surfaces` | `Dict[str, Dict[str, Any]]` | O | 서피스 ID를 키, 해당 서피스의 현재 데이터 모델을 값으로 하는 맵. |

---

### `A2uiClientMessageList`

`List[A2uiClientMessage]` — 메시지 목록 타입 별칭.

---

### `A2uiClientMessageListWrapper(StrictBaseModel)`

여러 클라이언트→서버 메시지를 하나의 객체로 감싸는 래퍼.

| 필드명 | 타입 | 필수 | 설명 |
|---|---|---|---|
| `messages` | `A2uiClientMessageList` | O | `A2uiClientMessage` 목록. |

## 동작 흐름

이 파일은 순수 데이터 컨테이너 모음이다. 클라이언트 렌더러는 사용자 인터랙션이 발생하면 `A2uiClientActionMessage`를 생성해 서버에 전달한다. 검증 또는 일반 오류 발생 시에는 `A2uiClientErrorMessage`를 생성한다. 여러 메시지를 일괄 전송할 때는 `A2uiClientMessageListWrapper`를 사용한다. `version` 필드에 `Literal[SPEC_VERSION]`을 사용해 잘못된 버전의 메시지가 역직렬화되는 것을 방지한다.
