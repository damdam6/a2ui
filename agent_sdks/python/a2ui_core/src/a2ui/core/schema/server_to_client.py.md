# agent_sdks/python/a2ui_core/src/a2ui/core/schema/server_to_client.py

## 개요

이 파일은 A2UI 프로토콜에서 서버가 클라이언트로 전송하는 모든 메시지 타입을 Pydantic 모델로 정의한다. 자동 생성 파일(`Auto-generated. Do not edit manually.`)로, 네 가지 메시지 유형(UI 서피스 생성, 컴포넌트 업데이트, 데이터 모델 업데이트, UI 서피스 삭제)을 각각 페이로드 클래스와 버전 포함 래퍼 클래스의 쌍으로 구현한다. 최종적으로 네 가지 래퍼를 `Union`으로 묶은 `A2uiMessage` 타입과 이를 리스트로 감싸는 `A2uiMessageListWrapper`를 통해 Pydantic 기반의 다형성 파싱을 지원한다.

## 의존성

### 외부 패키지
- `typing`: `Any`, `Dict`, `List`, `Literal`, `Optional`, `Union`
- `pydantic`: `BaseModel`, `Field`, `ConfigDict`

### 저장소 내부 모듈
- [`./common_types.py`](common_types.py.md) — `StrictBaseModel` (extra="forbid", populate_by_name=True 설정의 기반 모델)
- [`./constants.py`](constants.py.md) — `SPEC_VERSION` 프로토콜 버전 상수

## Exports

| 이름 | 종류 |
|---|---|
| `CreateSurface` | 클래스 (Pydantic 모델) |
| `CreateSurfaceMessage` | 클래스 (Pydantic 모델) |
| `UpdateComponents` | 클래스 (Pydantic 모델) |
| `UpdateComponentsMessage` | 클래스 (Pydantic 모델) |
| `UpdateDataModel` | 클래스 (Pydantic 모델) |
| `UpdateDataModelMessage` | 클래스 (Pydantic 모델) |
| `DeleteSurface` | 클래스 (Pydantic 모델) |
| `DeleteSurfaceMessage` | 클래스 (Pydantic 모델) |
| `A2uiMessage` | 타입 별칭 (`Union` of 4 message classes) |
| `A2uiMessageListWrapper` | 클래스 (Pydantic 모델) |

## 상세 명세

### `CreateSurface(StrictBaseModel)`

새 UI 서피스를 생성하도록 클라이언트에 지시하는 메시지 페이로드 본문.

| 필드 | 타입 | JSON alias | 기본값 | 설명 |
|---|---|---|---|---|
| `surface_id` | `str` | `surfaceId` | 필수 | 렌더링될 UI 서피스의 고유 식별자 |
| `catalog_id` | `str` | `catalogId` | 필수 | 카탈로그 고유 식별자. 충돌 방지를 위해 인터넷 도메인 접두사 권장 (예: `mycompany.com:somecatalog`) |
| `theme` | `Optional[Any]` | — | `None` | 서피스 테마 파라미터 dict. 카탈로그의 `theme` 스키마에 대해 유효성 검증되어야 함 |
| `send_data_model` | `Optional[bool]` | `sendDataModel` | `None` | `true`이면 클라이언트가 이 서피스를 생성한 서버에 보내는 모든 A2A 메시지 메타데이터에 전체 데이터 모델을 포함시킴; 기본값은 `false` |

### `CreateSurfaceMessage(StrictBaseModel)`

`createSurface` 동작을 감싸는 최상위 메시지 모델.

| 필드 | 타입 | JSON alias | 기본값 |
|---|---|---|---|
| `version` | `Literal[SPEC_VERSION]` | — | `SPEC_VERSION` (값이 상수와 일치해야 함) |
| `create_surface` | `CreateSurface` | `createSurface` | 필수 |

### `UpdateComponents(StrictBaseModel)`

기존 UI 서피스의 컴포넌트 목록 전체를 교체하도록 지시하는 메시지 페이로드 본문.

| 필드 | 타입 | JSON alias | 기본값 | 설명 |
|---|---|---|---|---|
| `surface_id` | `str` | `surfaceId` | 필수 | 업데이트할 UI 서피스 고유 식별자 |
| `components` | `List[Any]` | — | 필수 | 해당 서피스의 모든 UI 컴포넌트 목록 |

### `UpdateComponentsMessage(StrictBaseModel)`

`updateComponents` 동작을 감싸는 최상위 메시지 모델.

| 필드 | 타입 | JSON alias | 기본값 |
|---|---|---|---|
| `version` | `Literal[SPEC_VERSION]` | — | `SPEC_VERSION` |
| `update_components` | `UpdateComponents` | `updateComponents` | 필수 |

### `UpdateDataModel(StrictBaseModel)`

UI 서피스의 데이터 모델 일부 또는 전체를 갱신하거나 키를 제거하는 메시지 페이로드 본문.

| 필드 | 타입 | JSON alias | 기본값 | 설명 |
|---|---|---|---|---|
| `surface_id` | `str` | `surfaceId` | 필수 | 데이터 모델 업데이트가 적용될 UI 서피스 고유 식별자 |
| `path` | `Optional[str]` | — | `None` | 데이터 모델 내 위치를 가리키는 JSON Pointer (예: `/user/name`). 생략하거나 `/`이면 데이터 모델 전체를 참조 |
| `value` | `Optional[Any]` | — | `None` | 업데이트할 데이터. 존재하면 `path`의 값을 교체하거나 새로 생성. 생략하면 `path`의 키를 제거 |

### `UpdateDataModelMessage(StrictBaseModel)`

`updateDataModel` 동작을 감싸는 최상위 메시지 모델.

| 필드 | 타입 | JSON alias | 기본값 |
|---|---|---|---|
| `version` | `Literal[SPEC_VERSION]` | — | `SPEC_VERSION` |
| `update_data_model` | `UpdateDataModel` | `updateDataModel` | 필수 |

### `DeleteSurface(StrictBaseModel)`

UI 서피스 삭제를 지시하는 메시지 페이로드 본문.

| 필드 | 타입 | JSON alias | 기본값 |
|---|---|---|---|
| `surface_id` | `str` | `surfaceId` | 필수 |

### `DeleteSurfaceMessage(StrictBaseModel)`

`deleteSurface` 동작을 감싸는 최상위 메시지 모델.

| 필드 | 타입 | JSON alias | 기본값 |
|---|---|---|---|
| `version` | `Literal[SPEC_VERSION]` | — | `SPEC_VERSION` |
| `delete_surface` | `DeleteSurface` | `deleteSurface` | 필수 |

### `A2uiMessage` (타입 별칭)

`Union[CreateSurfaceMessage, UpdateComponentsMessage, UpdateDataModelMessage, DeleteSurfaceMessage]`로 정의된 타입 별칭. Pydantic이 이 Union을 선언 순서대로 시도하여 첫 번째로 유효성 검증에 성공하는 타입을 선택한다.

### `A2uiMessageListWrapper(StrictBaseModel)`

하나 이상의 A2UI 메시지를 담는 최상위 래퍼. 프로토콜 유효성 검증의 진입 타입으로 사용된다.

| 필드 | 타입 | 기본값 |
|---|---|---|
| `messages` | `List[A2uiMessage]` | 필수 |

## 동작 흐름

이 파일은 순수 데이터 모델 선언만으로 구성된다. 모든 클래스가 `StrictBaseModel`을 상속하므로 정의되지 않은 필드가 있는 입력은 파싱 시 `ValidationError`를 발생시킨다. `populate_by_name=True` 설정 덕분에 Python snake_case 이름과 JSON camelCase alias 양쪽으로 데이터를 받을 수 있다. 메시지 배열 전체를 `A2uiMessageListWrapper.model_validate({"messages": [...]})` 형태로 파싱하면 각 항목이 `A2uiMessage` Union을 통해 적절한 메시지 타입으로 디스패치된다. `version` 필드의 `Literal[SPEC_VERSION]` 타입은 버전 값이 정확히 일치해야 함을 강제하여 프로토콜 버전 불일치를 즉시 감지한다.
