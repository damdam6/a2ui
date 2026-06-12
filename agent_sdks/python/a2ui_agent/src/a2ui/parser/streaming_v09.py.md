# agent_sdks/python/a2ui_agent/src/a2ui/parser/streaming_v09.py

## 개요

A2UI 스트리밍 프로토콜 v0.9 사양에 특화된 파서 구현체를 제공한다. `A2uiStreamParser` 기반 클래스를 상속하여 v0.9 메시지 타입(`createSurface`, `updateComponents`, `updateDataModel`, `deleteSurface`)의 파싱·상태 관리·부분 메시지 조립 로직을 정의한다. v0.8과 달리 컴포넌트 구조가 평면(flat) 형식으로 표현되며, 데이터 모델 부분 업데이트(`_sniff_partial_data_model`) 기능을 추가로 지원한다.

## 의존성

### 외부 패키지
- `re` — JSON 버퍼 내 정규식 검색
- `json` — JSON 직렬화/역직렬화
- `typing` — `Any`, `List`, `Dict`, `Optional`, `Set` 타입 힌트

### 저장소 내부 모듈
- [`./streaming`](./streaming.py.md) — `A2uiStreamParser` 기반 클래스
- [`./response_part`](./response_part.py.md) — `ResponsePart` 타입
- [`./constants`](./constants.py.md) — 파서 레벨 메시지 타입 상수 (와일드카드 import)
- [`../schema/constants`](../schema/constants.py.md) — `VERSION_0_9`, `SURFACE_ID_KEY`, `CATALOG_COMPONENTS_KEY`

## Exports

- 클래스: `A2uiStreamParserV09`

## 상세 명세

### 클래스 `A2uiStreamParserV09(A2uiStreamParser)`

v0.9 프로토콜을 처리하는 스트리밍 파서 구현체이다.

#### `__init__(self, catalog=None)`

`super().__init__(catalog=catalog)`를 호출한 뒤, `_default_root_id = DEFAULT_ROOT_ID`를 설정한다. v0.9의 기본 루트 ID는 `"root"`이다.

#### 프로퍼티 `_placeholder_component -> Dict[str, Any]`

v0.9 평면 구조 형식의 플레이스홀더 컴포넌트를 반환한다: `{'component': 'Row', 'children': []}`. v0.8의 중첩 딕셔너리 방식과 달리 컴포넌트 이름을 문자열로 직접 사용한다.

#### 프로퍼티 `_data_model_msg_type -> str`

`MSG_TYPE_UPDATE_DATA_MODEL` 상수(`"updateDataModel"`)를 반환한다.

#### `is_protocol_msg(self, obj: Dict[str, Any]) -> bool`

`obj`에 `MSG_TYPE_CREATE_SURFACE`, `MSG_TYPE_UPDATE_COMPONENTS`, `MSG_TYPE_UPDATE_DATA_MODEL` 중 하나라도 키로 존재하면 `True`를 반환한다. v0.8과 달리 `deleteSurface`는 별도로 판별하지 않는다.

#### `_sniff_metadata(self) -> None`

아직 완성되지 않은 `_json_buffer`에서 정규식으로 메타데이터를 미리 추출한다. v0.8의 동명 메서드와 구조가 동일하지만, 감지하는 메시지 타입이 v0.9 기준(`createSurface`, `updateComponents`, `updateDataModel`)이다.

내부 헬퍼 `get_latest_value(key: str) -> Optional[str]`: `_json_buffer`를 역방향으로 순회하며 `"key": "value"` 패턴의 마지막 등장 값을 반환한다.

로직:
1. `surfaceId` 값으로 `self.surface_id` 갱신.
2. `root` 값이 있으면 `self.root_id` 갱신.
3. 각 v0.9 메시지 타입 키 문자열이 버퍼에 포함되면 `add_msg_type()`으로 등록.

#### `_handle_complete_object(self, obj: Dict[str, Any], sid: Optional[str], messages: List[ResponsePart]) -> bool`

완전히 파싱된 JSON 객체를 v0.9 방식으로 처리한다.

처리 순서:
1. `obj`가 `dict`가 아니면 `False` 반환.
2. `_validator`가 있으면 검증 실행.
3. `surfaceId`를 `obj` 또는 `updateComponents`/`createSurface` 서브값에서 추출하여 `self.surface_id` 갱신. `sid = self.surface_id or 'unknown'`으로 재설정.
4. `createSurface` 처리:
   - 서브값 dict에서 `root`를 갱신한다.
   - `_buffered_start_message = obj` 저장.
   - 아직 yield되지 않은 sid이면 즉시 방출 후 `_yielded_start_messages`, `_yielded_surfaces_set`에 추가, `_buffered_start_message = None`.
   - v0.8과 달리, 보류 중인 `_pending_messages[sid]`를 재처리하지 않고 버려버린다(`pop`만 수행). 이는 새 surface 시작 시 이전 상태를 완전히 초기화하기 위함이다.
   - `yield_reachable(messages)` 호출 후 `True` 반환.
5. `updateComponents` 처리:
   - `add_msg_type()` 등록.
   - `root` 키를 갱신한다 (`self.root_id or DEFAULT_ROOT_ID` 폴백 포함).
   - `components` 목록의 각 항목을 `_seen_components[comp['id']] = comp`에 저장.
   - `yield_reachable(messages, check_root=True, raise_on_orphans=False)` 호출.
   - `True` 반환.
6. `deleteSurface` 처리:
   - sid가 아직 `_yielded_start_messages`에 없으면 `_pending_messages`에 버퍼링 후 `True` 반환.
   - 이미 yield된 surface이면 `add_msg_type()` 후 `_yield_messages([obj], messages)`, `True` 반환.
7. `updateDataModel` 처리:
   - `add_msg_type()` 등록, `update_data_model()` 호출.
   - `_yield_messages([obj], messages)` 방출.
   - `True` 반환.
8. 인식되지 않는 메시지: `False` 반환.

#### `_construct_sniffed_data_model_message(self, active_msg_type: str, delta_msg_payload: Dict[str, Any]) -> Dict[str, Any]`

v0.9 부분 데이터 모델 메시지를 생성한다. 반환값: `{'version': 'v0.9', active_msg_type: delta_msg_payload}`. v0.8에는 없는 메서드이다.

#### `_sniff_partial_data_model(self, messages: List[ResponsePart]) -> None`

스트리밍 중 아직 완성되지 않은 `updateDataModel` 메시지에서 점진적으로 변경된 데이터 모델 값을 추출해 방출한다.

로직:
1. `_json_buffer`에 `MSG_TYPE_UPDATE_DATA_MODEL` 키 문자열이 없으면 바로 반환.
2. `_brace_stack`을 역순으로 순회하며 `{` 타입의 위치를 찾는다.
3. 각 위치에서 `_json_buffer[start_idx:]`를 추출하고 `_fix_json()`으로 보정 후 `json.loads()` 시도. 실패 시 마지막 쉼표를 반복 제거하며 재시도한다.
4. 파싱된 객체에 `updateDataModel` 키가 있고 그 값 dict에 `'value'` 키가 있으면, `value` dict와 `_yielded_data_model` 간의 델타를 계산한다.
5. 델타가 있으면 `_construct_sniffed_data_model_message()`로 메시지를 조립하고 `_yield_messages(... strict_integrity=False)`로 방출한다.
6. 중복 방출 방지를 위해 `_yielded_data_model.update(delta)`로 즉시 반영한다.

#### `_construct_partial_message(self, processed_components: List[Dict[str, Any]], active_msg_type: str) -> Dict[str, Any]`

스트리밍 중 부분 컴포넌트 메시지를 v0.9 형식으로 조립한다. `surface_id`가 있으면 페이로드에 포함한다. 반환값: `{'version': 'v0.9', MSG_TYPE_UPDATE_COMPONENTS: {CATALOG_COMPONENTS_KEY: processed_components, [SURFACE_ID_KEY: self.surface_id]}}`.

#### 프로퍼티 `_yielded_surfaces_set -> Set[str]`

`_yielded_create_surfaces` 속성이 없으면 지연 초기화하여 반환한다. `createSurface` 메시지를 이미 방출한 surface ID 집합이다. v0.8의 `_yielded_begin_rendering_surfaces`에 대응한다.

#### `_get_active_msg_type_for_components(self) -> Optional[str]`

컴포넌트 업데이트를 감쌀 메시지 타입을 결정한다.
1. `_active_msg_type`이 설정되어 있으면 반환.
2. `_msg_types`에서 `MSG_TYPE_UPDATE_COMPONENTS` 또는 `MSG_TYPE_CREATE_SURFACE`를 찾아 저장 후 반환.
3. 아무것도 없으면 `_msg_types[0]` 또는 `None` 반환.

#### `_deduplicate_data_model(self, m: Dict[str, Any], strict_integrity: bool) -> bool`

v0.9 `updateDataModel` 메시지의 중복 방출을 방지한다. v0.8 버전과의 차이점: v0.9의 데이터 모델 페이로드는 리스트/중첩 contents 구조가 아닌 평면 딕셔너리이며, `surfaceId`와 `root` 키는 비교에서 제외한다.

로직:
1. 메시지에 `MSG_TYPE_UPDATE_DATA_MODEL` 키가 없으면 `True` 반환.
2. 값 dict를 순회하며 `surfaceId`와 `root`를 제외한 키에서 변경 여부를 확인.
3. 변경이 없고 `strict_integrity=True`이면 `False` 반환.
4. 변경된 키를 `_yielded_data_model`에 업데이트 후 `True` 반환.

## 동작 흐름

JSON 스트림 청크가 기반 클래스를 통해 `_json_buffer`에 누적되면 `_sniff_metadata()`로 메타데이터를 추출하고, `_sniff_partial_data_model()`로 데이터 모델 부분 업데이트를 즉시 방출한다. 완전한 JSON 객체가 완성되면 `_handle_complete_object()`로 분기 처리된다. `createSurface` 수신 즉시 방출하며, 이후 `updateComponents`가 올 때마다 도달 가능한 컴포넌트를 골라 점진적으로 방출한다. v0.8과 달리 `createSurface` 이전에 도착한 보류 메시지는 버려지며, 데이터 모델 업데이트는 스트리밍 중에도 `value` 딕셔너리 델타 방식으로 점진 방출된다.
