# agent_sdks/python/a2ui_agent/src/a2ui/parser/streaming_v08.py

## 개요

A2UI 스트리밍 프로토콜 v0.8 사양에 특화된 파서 구현체를 제공한다. `A2uiStreamParser` 기반 클래스를 상속하여 v0.8 메시지 타입(`beginRendering`, `surfaceUpdate`, `dataModelUpdate`, `deleteSurface`)의 파싱·상태 관리·부분 메시지 조립 로직을 정의한다. 스트리밍 도중 도착하는 JSON 청크를 점진적으로 처리하여 렌더링 가능한 시점에 즉시 메시지를 방출(yield)하는 역할을 담당한다.

## 의존성

### 외부 패키지
- `re` — JSON 버퍼 내 정규식 검색
- `json` — JSON 직렬화/역직렬화
- `typing` — `Any`, `List`, `Dict`, `Optional`, `Set` 타입 힌트

### 저장소 내부 모듈
- [`./streaming`](./streaming.py.md) — `A2uiStreamParser` 기반 클래스
- [`./response_part`](./response_part.py.md) — `ResponsePart` 타입
- [`./constants`](./constants.py.md) — 파서 레벨 메시지 타입 상수 (와일드카드 import)
- [`../schema/constants`](../schema/constants.py.md) — `VERSION_0_8`, `SURFACE_ID_KEY`, `CATALOG_COMPONENTS_KEY`

## Exports

- 클래스: `A2uiStreamParserV08`

## 상세 명세

### 클래스 `A2uiStreamParserV08(A2uiStreamParser)`

v0.8 프로토콜을 처리하는 스트리밍 파서 구현체이다.

#### `__init__(self, catalog=None)`

`super().__init__(catalog=catalog)`를 호출한 뒤, `_yielded_begin_rendering_surfaces: Set[str]`를 빈 집합으로 초기화한다. 이 집합은 이미 `beginRendering` 메시지를 방출한 surface ID를 추적한다.

#### 프로퍼티 `_placeholder_component -> Dict[str, Any]`

파싱이 진행 중일 때 루트 컴포넌트가 아직 완성되지 않은 경우 사용할 플레이스홀더 컴포넌트 딕셔너리를 반환한다. v0.8에서는 중첩 구조 형식으로 `{'component': {'Row': {'children': {'explicitList': []}}}}` 를 반환한다.

#### 프로퍼티 `_yielded_surfaces_set -> Set[str]`

버전별 "이미 begin 메시지를 방출한 surface ID" 집합에 대한 접근자로, `_yielded_begin_rendering_surfaces`를 반환한다.

#### `is_protocol_msg(self, obj: Dict[str, Any]) -> bool`

`obj` 딕셔너리 안에 `MSG_TYPE_BEGIN_RENDERING`, `MSG_TYPE_SURFACE_UPDATE`, `MSG_TYPE_DATA_MODEL_UPDATE`, `MSG_TYPE_DELETE_SURFACE` 중 하나라도 키로 존재하면 `True`를 반환한다.

#### 프로퍼티 `_data_model_msg_type -> str`

`MSG_TYPE_DATA_MODEL_UPDATE` 상수(`"dataModelUpdate"`)를 반환한다. 기반 클래스에서 데이터 모델 메시지 종류를 판별할 때 사용된다.

#### `_sniff_metadata(self) -> None`

아직 완성되지 않은 `_json_buffer`에서 정규식을 이용해 메타데이터를 미리 추출하는 메서드이다.

내부 헬퍼 `get_latest_value(key: str) -> Optional[str]`를 정의한다. 이 헬퍼는 `_json_buffer`를 역방향(`rfind`)으로 순회하며 `"key": "value"` 패턴을 정규식으로 찾아 마지막으로 등장하는 값을 반환한다.

로직 흐름:
1. `get_latest_value('surfaceId')`로 `self.surface_id`를 갱신한다.
2. `get_latest_value('root')`가 `None`이 아니면 `self.root_id`를 갱신한다.
3. 각 메시지 타입 문자열(`"beginRendering":` 등)이 버퍼에 포함되어 있으면 해당 타입을 `add_msg_type()`으로 등록한다.

#### `_handle_complete_object(self, obj: Dict[str, Any], sid: Optional[str], messages: List[ResponsePart]) -> bool`

완전히 파싱된 JSON 객체 `obj`를 처리한다.

처리 순서:
1. `obj`가 `dict`가 아니면 `False` 반환.
2. `_validator`가 있으면 `validate(obj, root_id=sid, strict_integrity=False)` 호출.
3. `obj`에서 `surfaceId` 키 또는 메시지별 서브딕셔너리에서 `surface_id`를 갱신한다. `surfaceUpdate`, `beginRendering`, `deleteSurface`의 값이 dict이면 그 안의 `surfaceId`를 우선 적용한다. `deleteSurface` 값이 str이면 그 자체를 surface_id로 사용한다.
4. `self.surface_id`를 최종 결정된 값으로 설정하고, `sid = self.surface_id or 'unknown'`으로 재설정한다.
5. `deleteSurface` 키가 있고 이미 yield된 surface이거나 `_buffered_start_message`가 있으면 `_delete_surface(sid)` 호출.
6. `sid`가 `_deleted_surfaces`에 포함되어 있으면 `True` 반환(더 이상 처리 안 함).
7. `surfaceUpdate` 또는 `deleteSurface` 메시지이고 해당 surface가 아직 yield되지 않았으며 `_buffered_start_message`도 없으면, 메시지를 `_pending_messages[sid]`에 버퍼링하고 `True` 반환.
8. `beginRendering` 처리:
   - 서브값에서 `surfaceId`와 `root`를 갱신한다.
   - `_buffered_start_message = obj`로 저장한다.
   - 해당 sid가 아직 yield되지 않았으면 즉시 `_yield_messages([obj], messages)` 호출 후 `_yielded_start_messages`와 `_yielded_surfaces_set`에 sid를 추가, `_buffered_start_message = None`으로 초기화한다.
   - 보류 중인 `_pending_messages[sid]`가 있으면 각 메시지를 재귀 호출(`_handle_complete_object`)로 처리한다.
   - `yield_reachable(messages)`를 호출한다.
   - `True` 반환.
9. `surfaceUpdate` 처리:
   - `MSG_TYPE_SURFACE_UPDATE`를 `add_msg_type()`으로 등록.
   - `components` 목록을 순회하며 `id` 키가 있는 dict를 `_seen_components[comp['id']] = comp`에 저장.
   - `yield_reachable(messages, check_root=True, raise_on_orphans=False)` 호출.
   - `True` 반환.
10. `dataModelUpdate` 처리:
    - `add_msg_type()` 등록 후 `update_data_model()` 호출.
    - `_yield_messages([obj], messages)`로 원본 메시지도 방출.
    - `yield_reachable(messages, check_root=False, raise_on_orphans=False)` 호출.
    - `True` 반환.
11. `deleteSurface` 처리: `_yield_messages([obj], messages)` 후 `True` 반환.
12. 알 수 없는 메시지: `_yield_messages([obj], messages)` 후 `True` 반환.

#### `_construct_partial_message(self, processed_components: List[Dict[str, Any]], active_msg_type: str) -> Dict[str, Any]`

아직 스트리밍 중인 컴포넌트 목록을 v0.8 형식의 부분 메시지로 조립한다. 반환 구조: `{MSG_TYPE_SURFACE_UPDATE: {SURFACE_ID_KEY: self.surface_id, CATALOG_COMPONENTS_KEY: processed_components}}`. `active_msg_type` 인수는 현재 구현에서 사용되지 않는다(항상 `surfaceUpdate` 형식으로 반환).

#### `_get_active_msg_type_for_components(self) -> Optional[str]`

컴포넌트 업데이트를 감쌀 메시지 타입을 결정한다.
1. `_active_msg_type`이 이미 설정되어 있으면 그대로 반환.
2. `_msg_types` 목록에서 `MSG_TYPE_SURFACE_UPDATE` 또는 `MSG_TYPE_BEGIN_RENDERING`을 찾으면 해당 타입을 `_active_msg_type`으로 저장 후 반환.
3. 아무것도 없으면 `_msg_types[0]` 또는 `None` 반환.

#### `_deduplicate_data_model(self, m: Dict[str, Any], strict_integrity: bool) -> bool`

v0.8 `dataModelUpdate` 메시지의 중복 방출을 방지한다.

로직:
1. 메시지에 `MSG_TYPE_DATA_MODEL_UPDATE` 키가 없으면 `True` 반환(중복 아님으로 취급).
2. `contents` 필드를 파싱한다. 리스트이면 각 항목에서 `key`와 `valueString | valueNumber | valueBoolean | valueMap` 중 하나를 추출해 `contents_dict`를 구성한다. 딕셔너리이면 그대로 사용한다.
3. `contents_dict`가 비어있지 않으면 현재 `_yielded_data_model`과 비교해 하나라도 변경된 값이 있는지 확인(`is_new` 플래그).
4. `is_new`가 `False`이고 `strict_integrity=True`이면 `False` 반환(중복 방출 차단).
5. 변경된 값을 `_yielded_data_model.update(contents_dict)`로 기록 후 `True` 반환.

## 동작 흐름

JSON 스트림이 청크 단위로 도착하면 기반 클래스(`A2uiStreamParser`)가 청크를 `_json_buffer`에 누적하고 `_sniff_metadata()`로 메타데이터를 미리 추출한다. 완전한 JSON 객체가 감지되면 `_handle_complete_object()`를 호출하여 메시지 타입별로 분기 처리한다. `beginRendering`이 완성되는 순간 즉시 방출하고, 이후 도착하는 `surfaceUpdate`는 도달 가능한 컴포넌트만 골라(`yield_reachable`) 점진적으로 방출한다. surface가 아직 시작되지 않은 상태에서 `surfaceUpdate`가 먼저 도착하면 `_pending_messages`에 보류했다가 `beginRendering` 도착 시 처리한다.
