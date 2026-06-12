# agent_sdks/python/a2ui_core/src/a2ui/core/validating/validator.py

## 개요

A2UI 프로토콜 페이로드 전체에 대한 종합 유효성 검증을 수행하는 최상위 오케스트레이터 파일이다. 프로토콜 봉투(envelope) 구조 검증, 컴포넌트 스키마 검증, 참조 무결성 검증, 위상 분석을 순서대로 실행하며, 수집된 모든 오류를 `A2uiValidatorError`로 집약하여 발생시킨다. `ValidationConfig`로 동작 허용 범위를 조정할 수 있다.

## 의존성

### 외부 패키지
- `re`
- `typing`: `Any`, `Dict`, `Iterator`, `List`, `Optional`, `Set`, `Tuple`, `Union`
- `pydantic`: `BaseModel`, `ValidationError`

### 저장소 내부 모듈
- [`../schema/__init__.py`](../schema/__init__.py.md) — `A2uiMessageListWrapper` (server_to_client.py에서 재노출)
- [`../schema/constants.py`](../schema/constants.py.md) — `MSG_TYPE_CREATE_SURFACE`, `MSG_TYPE_UPDATE_COMPONENTS`, `MSG_TYPE_UPDATE_DATA_MODEL`, `MSG_TYPE_DELETE_SURFACE`, `CATALOG_COMPONENTS_KEY`, `THEME_KEY`
- [`./integrity_checker.py`](integrity_checker.py.md) — `validate_component_integrity`, `validate_recursion_and_paths`
- [`./topology_analyzer.py`](topology_analyzer.py.md) — `analyze_topology`
- [`./catalog_validator.py`](catalog_validator.py.md) — `CatalogValidator`

## Exports

| 이름 | 종류 |
|---|---|
| `A2uiValidatorError` | 예외 클래스 |
| `ValidationConfig` | 클래스 (Pydantic `BaseModel`) |
| `A2uiValidator` | 클래스 |

## 상세 명세

### `A2uiValidatorError(ValueError)`

A2UI 카탈로그 페이로드 유효성 검증 실패 시 발생하는 커스텀 예외 클래스다. `ValueError`의 직접 서브클래스이며 추가 로직이나 필드 없이 타입 식별만을 위해 존재한다.

---

### `ValidationConfig(BaseModel)`

A2UI 페이로드 및 컴포넌트 검증의 동작 옵션을 담는 Pydantic 설정 모델이다. `StrictBaseModel`이 아닌 일반 `BaseModel`을 상속한다.

| 필드 | 타입 | 기본값 | 설명 |
|---|---|---|---|
| `allow_orphan_components` | `bool` | `False` | `True`이면 루트에서 도달할 수 없는 고아 컴포넌트를 오류 없이 허용 |
| `allow_dangling_references` | `bool` | `False` | `True`이면 컴포넌트 목록에 존재하지 않는 ID를 참조해도 오류 없이 허용 (클라이언트에 이미 존재하는 컴포넌트를 참조하는 증분 업데이트에 사용) |

---

### `A2uiValidator`

A2UI JSON 페이로드를 카탈로그 스키마와 레이아웃 무결성에 대해 검증하는 메인 클래스. 생성자(`__init__`) 없이 상태를 갖지 않는 메서드만으로 구성된다.

#### `validate_protocol_envelope(self, messages: List[Dict[str, Any]]) -> None`

메시지 목록의 프로토콜 봉투 구조를 검증한다.

동작:
1. `struct_errors = []`를 초기화한다.
2. `messages`의 각 `msg`에 대해: `msg`가 딕셔너리가 아니면 `"Message must be an object"`를 추가하고, `"version"` 키가 없으면 `"'version' is a required property"`를 `struct_errors`에 추가한다.
3. `try` 블록에서 `A2uiMessageListWrapper.model_validate({"messages": messages})`로 Pydantic 전체 구조 검증을 수행한다. 성공하면 `for msg in messages: validate_recursion_and_paths(messages)` 루프를 실행한다(각 메시지 순회이나 인수는 `messages` 전체 리스트임에 주의). `ValidationError` 발생 시 `_format_validation_errors(e, messages)`의 결과를 `struct_errors.extend(...)`로 추가한다.
4. `struct_errors`가 비어 있지 않으면 `"\n".join(struct_errors)`로 이어 붙여 `A2uiValidatorError`를 발생시킨다.

#### `_format_validation_errors(self, error: ValidationError, messages: List[Dict[str, Any]]) -> List[str]` (비공개)

Pydantic `ValidationError`의 오류 항목을 필터링하여 사람이 읽기 좋은 문자열 목록으로 변환한다. Union 타입 파싱 시 발생하는 무관한 분기의 오류를 제거한다.

동작: `error.errors()`를 순회하며 각 `err` 항목에 대해:
1. `err.get("loc", [])`를 `loc`로, 문자열 목록으로 변환한 것을 `loc_parts`로 사용한다.
2. `len(loc) >= 3`이고 `loc[0] == "messages"`, `loc[1]`이 `int`이면: 해당 인덱스의 메시지가 존재하고 딕셔너리이면 그 메시지가 `MSG_TYPE_CREATE_SURFACE`(`"createSurface"`), `MSG_TYPE_UPDATE_COMPONENTS`(`"updateComponents"`), `MSG_TYPE_UPDATE_DATA_MODEL`(`"updateDataModel"`), `MSG_TYPE_DELETE_SURFACE`(`"deleteSurface"`) 중 하나를 키로 포함하는지(`is_recognized_message_type`) 확인한다.
3. 인식된 메시지 타입이면 `loc_parts[2]`(Pydantic Union 분기 이름)를 확인한다. 다음 네 가지 조건 중 하나에 해당하면 `continue`로 해당 오류를 건너뛴다:
   - `branch == "CreateSurfaceMessage"` 이고 메시지에 `MSG_TYPE_CREATE_SURFACE` 키가 없는 경우
   - `branch == "UpdateComponentsMessage"` 이고 메시지에 `MSG_TYPE_UPDATE_COMPONENTS` 키가 없는 경우
   - `branch == "UpdateDataModelMessage"` 이고 메시지에 `MSG_TYPE_UPDATE_DATA_MODEL` 키가 없는 경우
   - `branch == "DeleteSurfaceMessage"` 이고 메시지에 `MSG_TYPE_DELETE_SURFACE` 키가 없는 경우
4. 필터링을 통과한 오류는 `f"{'.'.join(loc_parts)}: {err.get('msg', 'Validation failed')}"` 형태로 `formatted_errors`에 추가한다.
5. `formatted_errors`를 반환한다.

#### `validate_components(self, catalog_validator, components, config) -> None`

**시그니처**: `validate_components(self, catalog_validator: CatalogValidator, components: List[Dict[str, Any]], config: ValidationConfig = ValidationConfig()) -> None`

컴포넌트 목록에 대해 스키마 검증, 참조 무결성 검증, 위상 분석을 순서대로 수행한다. 모든 오류를 수집하여 한 번에 발생시킨다.

동작:
1. `errors = []`를 초기화한다.
2. `components`가 비어 있지 않으면: 각 컴포넌트 `c`에 대해 개별적으로 `catalog_validator.validate_components([c])`를 호출하고, `Exception` 발생 시 `errors.append(ce)`로 수집한다. 이렇게 하면 첫 번째 오류에서 단락(short-circuit)하지 않고 모든 스키마 오류를 모을 수 있다.
3. `errors`가 비어 있으면 (스키마 오류 없음):
   - `ref_fields = catalog_validator.extract_ref_fields()`를 호출한다.
   - `validate_component_integrity(components, ref_fields, allow_dangling_references=config.allow_dangling_references)`를 호출한다.
   - `analyze_topology(components, ref_fields, allow_orphan_components=config.allow_orphan_components)`를 호출한다.
   - 예외 발생 시 `errors.append(e)`로 수집한다.
4. `errors`가 비어 있지 않으면 `"\n".join(str(err) for err in errors)`로 이어 붙여 `A2uiValidatorError`를 발생시킨다.

#### `validate(self, catalog_validator, a2ui_payload, config) -> None`

**시그니처**: `validate(self, catalog_validator: CatalogValidator, a2ui_payload: Union[Dict[str, Any], List[Any]], config: Optional[ValidationConfig] = None) -> None`

A2UI 페이로드 전체에 대한 최상위 진입 메서드.

동작:
1. `config`가 `None`이면 `ValidationConfig(allow_orphan_components=False, allow_dangling_references=False)`를 새로 생성한다.
2. `a2ui_payload`가 `list`이면 그대로, `dict`이면 단일 요소 리스트 `[a2ui_payload]`로 정규화하여 `messages`를 만든다.
3. `errors = []`를 초기화한다.
4. `self.validate_protocol_envelope(messages)`를 호출하고, 예외 발생 시 `errors.append(e)`로 수집한다.
5. `messages`를 순회하며 각 `msg`가 딕셔너리인 경우 `try` 블록에서:
   - `MSG_TYPE_CREATE_SURFACE`(`"createSurface"`) 키가 `msg`에 있으면: `msg[MSG_TYPE_CREATE_SURFACE].get(THEME_KEY)`로 테마를 추출하고, `theme`이 truthy이면 `catalog_validator.validate_theme(theme)`을 호출한다.
   - `MSG_TYPE_UPDATE_COMPONENTS`(`"updateComponents"`) 키가 `msg`에 있으면: `msg[MSG_TYPE_UPDATE_COMPONENTS].get(CATALOG_COMPONENTS_KEY)`로 컴포넌트 목록을 추출하고, `isinstance(components, list)`이면 `self.validate_components(catalog_validator, components, config=config)`를 호출한다.
   - 예외 발생 시 `errors.append(e)`로 수집한다.
6. `errors`가 비어 있지 않으면 `"\n".join(str(err) for err in errors)`로 이어 붙여 `A2uiValidatorError`를 발생시킨다.

## 동작 흐름

`validate`가 최상위 진입점으로 다음 세 단계를 실행한다:

1. **프로토콜 봉투 검증** (`validate_protocol_envelope`): Pydantic(`A2uiMessageListWrapper`)으로 메시지 구조 확인, 재귀 깊이 및 경로 구문 검사(`validate_recursion_and_paths`)
2. **메시지 타입별 분기**:
   - `createSurface` → `catalog_validator.validate_theme()`으로 테마 스키마 검증
   - `updateComponents` → `validate_components()`로 개별 컴포넌트 스키마 검증 + `validate_component_integrity()`로 무결성 검증 + `analyze_topology()`로 위상 분석
3. **오류 집약**: 각 단계의 예외를 `errors` 리스트에 누적한 후 한꺼번에 `A2uiValidatorError`로 발생시킨다.

프로토콜 봉투 검증 실패가 있어도 메시지 레벨 검증이 계속 진행되어, 가능한 한 많은 오류를 한 번에 보고한다. 반대로 `validate_components` 내부에서는 스키마 오류가 있으면 무결성/위상 검증을 건너뛰어 이미 실패한 컴포넌트에 대한 불필요한 추가 검사를 방지한다.
