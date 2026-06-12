# agent_sdks/python/a2ui_core/tests/test_schema.py

## 개요

`a2ui.core.schema` 패키지에서 생성된 Pydantic 스키마 모델들의 파싱 및 직렬화 동작을 pytest로 검증하는 테스트 파일이다. 유효한 딕셔너리가 올바르게 파싱되고 필드에 접근 가능한지, 잘못된 `version` 값에서 `ValidationError`가 발생하는지, camelCase alias와 snake_case 양쪽으로 모델을 생성자 인자로 구성할 수 있는지를 확인한다. 픽스처 없이 테스트 함수 간 공유 상태 없이 독립적으로 실행된다.

## 의존성

### 외부 패키지
- `pytest` — 테스트 프레임워크 및 `pytest.raises`
- `pydantic.ValidationError` — 유효성 검사 실패 예외 타입
- `typing.get_args` — import되어 있으나 테스트 본문에서 직접 사용하지 않음

### 저장소 내부 모듈
- [`agent_sdks/python/a2ui_core/src/a2ui/core/schema/__init__.py`](../src/a2ui/core/schema/__init__.py.md) — 아래 클래스를 export:
  - `A2uiClientMessageListWrapper`
  - `A2uiClientActionMessage`
  - `A2uiClientErrorMessage`
  - `A2uiClientDataModel`
  - `A2uiValidationError`
  - `A2uiGenericError`
  - `A2uiMessage`
  - `DeleteSurfaceMessage`
  - `CreateSurface` (테스트 함수 내에서 로컬 import)

## Exports

테스트 파일이므로 공개 export 없음.

## 동작 흐름 (테스트 케이스 상세)

### `test_valid_action_message`

`A2uiClientActionMessage.model_validate`로 action 메시지를 파싱한다. 픽스처/모킹 없음.

입력 딕셔너리: `version="v0.9"`, `action.name="submit"`, `action.surfaceId="s1"`, `action.sourceComponentId="c1"`, `action.timestamp="2026-06-02T23:37:16Z"`, `action.context={"foo": "bar"}`.

검증:
- `msg.version == "v0.9"`
- `msg.action.name == "submit"`
- `msg.action.context == {"foo": "bar"}`

---

### `test_valid_validation_error_message`

`A2uiClientErrorMessage.model_validate`로 `VALIDATION_FAILED` 에러 메시지를 파싱한다. 픽스처/모킹 없음.

입력: `version="v0.9"`, `error.code="VALIDATION_FAILED"`, `error.surfaceId="s1"`, `error.path="/components/0/text"`, `error.message="Too short"`.

검증:
- `msg.version == "v0.9"`
- `isinstance(msg.error, A2uiValidationError)` — discriminated union에서 올바른 타입으로 판별
- `msg.error.code == "VALIDATION_FAILED"`
- `msg.error.path == "/components/0/text"`

---

### `test_valid_generic_error_message`

`A2uiClientErrorMessage.model_validate`로 `INTERNAL_ERROR` 일반 에러 메시지를 파싱한다. 픽스처/모킹 없음.

입력: `version="v0.9"`, `error.code="INTERNAL_ERROR"`, `error.message="Something went wrong"`, `error.surfaceId="s1"`.

검증:
- `msg.version == "v0.9"`
- `isinstance(msg.error, A2uiGenericError)` — `A2uiValidationError`가 아닌 `A2uiGenericError`로 판별됨
- `msg.error.code == "INTERNAL_ERROR"`
- `msg.error.message == "Something went wrong"`

---

### `test_valid_data_model_message`

`A2uiClientDataModel.model_validate`로 복수 surface 데이터를 포함한 메시지를 파싱한다. 픽스처/모킹 없음.

입력: `version="v0.9"`, `surfaces={"s1": {"user": "Alice"}, "s2": {"cart": []}}`.

검증:
- `msg.version == "v0.9"`
- `msg.surfaces["s1"] == {"user": "Alice"}`
- `msg.surfaces["s2"] == {"cart": []}`

---

### `test_fails_on_invalid_version`

지원되지 않는 버전 값 `"v0.8"`을 포함한 action 메시지가 `ValidationError`를 발생시키는지 확인한다. 픽스처/모킹 없음.

입력: `test_valid_action_message`와 동일한 구조이나 `version="v0.8"`. `pytest.raises(ValidationError)`로 예외 발생을 단언한다.

---

### `test_valid_delete_surface_server_message`

`DeleteSurfaceMessage.model_validate`로 서버→클라이언트 deleteSurface 메시지를 파싱한다. 픽스처/모킹 없음.

입력: `version="v0.9"`, `deleteSurface.surfaceId="surface-1"`.

검증:
- `parsed.version == "v0.9"`
- `parsed.delete_surface.surface_id == "surface-1"` — camelCase alias `surfaceId`가 snake_case `surface_id`로 매핑되는지 확인

---

### `test_seamless_programmatic_construction_snake_or_alias`

`a2ui.core.schema.CreateSurface` 모델을 snake_case와 camelCase alias 두 가지 방식으로 구성자 인자를 통해 생성할 수 있음을 검증한다. 테스트 함수 내에서 `from a2ui.core.schema import CreateSurface`로 로컬 import한다.

**케이스 1 — snake_case 키워드 인자:**
`CreateSurface(surface_id="surf-snake", catalog_id="cat-snake")`로 생성.
- `obj_snake.surface_id == "surf-snake"`, `obj_snake.catalog_id == "cat-snake"`
- `obj_snake.model_dump(by_alias=True)["surfaceId"] == "surf-snake"` — 직렬화 시 camelCase alias로 출력됨

**케이스 2 — camelCase alias 키워드 인자:**
`CreateSurface(surfaceId="surf-alias", catalogId="cat-alias")`로 생성.
- `obj_alias.surface_id == "surf-alias"`, `obj_alias.catalog_id == "cat-alias"`
- `obj_alias.model_dump(by_alias=True)["surfaceId"] == "surf-alias"`

이 테스트는 Pydantic 모델이 `populate_by_name=True` 설정(또는 동등한 설정)으로 정의되어 snake_case와 alias 양쪽을 생성자 인자로 허용하는지 확인한다.
