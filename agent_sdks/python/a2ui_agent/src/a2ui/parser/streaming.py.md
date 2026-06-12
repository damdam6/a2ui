# agent_sdks/python/a2ui_agent/src/a2ui/parser/streaming.py

## 개요

LLM 스트림에서 점진적으로 수신되는 텍스트 청크를 처리하여 완성 전 단계에서도 A2UI 컴포넌트를 즉시 yield하는 스트리밍 파서 기반 클래스 `A2uiStreamParser`를 정의한다. 클래스는 팩토리 패턴(`__new__` 오버라이드)을 사용하여 카탈로그 버전에 따라 `A2uiStreamParserV08` 또는 `A2uiStreamParserV09` 인스턴스를 자동으로 반환한다. 공통 버퍼 관리, JSON 단편 복구(`_fix_json`), 컴포넌트 토폴로지 처리(`yield_reachable`, `_process_component_topology`), 부분 데이터 모델 스니핑 등 버전 공통 로직을 제공하며, 버전별 세부 동작은 추상 메서드를 통해 서브클래스에 위임한다.

## 의존성

- 외부 패키지:
  - `copy` (표준 라이브러리)
  - `json` (표준 라이브러리)
  - `logging` (표준 라이브러리)
  - `re` (표준 라이브러리)
  - `typing` (표준 라이브러리) — `Any`, `List`, `Dict`, `Optional`, `Set`, `TYPE_CHECKING`
- 내부 모듈:
  - [`./constants.py`](./constants.py.md) — `MSG_TYPE_*` 상수 전체 (`from .constants import *`)
  - `../schema/constants.py` — `VERSION_0_9`, `VERSION_0_8`, `A2UI_OPEN_TAG`, `A2UI_CLOSE_TAG`, `SURFACE_ID_KEY`, `CATALOG_COMPONENTS_KEY`
  - `../schema/validator.py` — `analyze_topology`, `extract_component_ref_fields`, `extract_component_required_fields`, `A2uiValidator`
  - [`./response_part.py`](./response_part.py.md) — `ResponsePart`
  - (TYPE_CHECKING 전용) `../schema/catalog.py` — `A2uiCatalog`
  - (런타임 지연 임포트) `./streaming_v09.py` — `A2uiStreamParserV09`
  - (런타임 지연 임포트) `./streaming_v08.py` — `A2uiStreamParserV08`

## Exports

- `A2uiStreamParser` — 클래스 (팩토리 + 기반 클래스)

## 상세 명세

### `A2uiStreamParser`

#### 팩토리 동작 (`__new__`)

시그니처: `__new__(cls, catalog: "A2uiCatalog" = None, *args, **kwargs)`

`cls`가 `A2uiStreamParser` 자체일 때만 팩토리 로직이 실행된다. `catalog.version`을 조회하여 `VERSION_0_9`이면 `A2uiStreamParserV09` 인스턴스를, 그 외에는 `A2uiStreamParserV08` 인스턴스를 반환한다. 서브클래스에서 `super().__new__(cls)`로 호출 시에는 일반 객체 생성 경로를 따른다. 두 버전별 파서 모듈은 순환 참조 방지를 위해 이 시점에 지연 임포트된다.

#### `__init__(self, catalog: "A2uiCatalog" = None)`

인스턴스 상태를 초기화한다. 서브클래스에서 `super().__init__`으로 호출해야 한다.

**초기화되는 주요 속성:**

| 속성 | 타입 | 초기값 | 역할 |
|------|------|--------|------|
| `_version` | `Optional[str]` | `catalog.version` or `None` | 프로토콜 버전 식별 |
| `_cuttable_keys` | `frozenset` | `catalog.cuttable_keys` or `frozenset()` | 부분 문자열 자동 닫기 허용 키 목록 |
| `_ref_fields_map` | `Dict` | `extract_component_ref_fields(catalog)` or `{}` | 컴포넌트 참조 필드 매핑 |
| `_required_fields_map` | `Dict` | `extract_component_required_fields(catalog)` or `{}` | 컴포넌트 필수 필드 매핑 |
| `_validator` | `A2uiValidator` or `None` | 카탈로그 존재 시 생성 | 메시지 검증기 |
| `_found_delimiter` | `bool` | `False` | 현재 `A2UI_OPEN_TAG` 내부에 있는지 여부 |
| `_buffer` | `str` | `""` | 아직 처리되지 않은 원시 텍스트 버퍼 |
| `_json_buffer` | `str` | `""` | A2UI 태그 내부 JSON 단편 누적 버퍼 |
| `_brace_stack` | `List[Tuple[str, int]]` | `[]` | `("{", 시작인덱스)` 또는 `("[", 시작인덱스)` 스택 |
| `_brace_count` | `int` | `0` | 현재 열린 중괄호/대괄호 총 깊이 |
| `_in_top_level_list` | `bool` | `False` | 최상위 JSON 배열 내부인지 여부 |
| `_in_string` | `bool` | `False` | 현재 JSON 문자열 파싱 중인지 여부 |
| `_string_escaped` | `bool` | `False` | 다음 문자가 이스케이프된 문자인지 여부 |
| `_seen_components` | `Dict[str, Dict]` | `{}` | 스트림에서 파싱된 컴포넌트 캐시 (cid → comp) |
| `_yielded_data_model` | `Dict[str, Any]` | `{}` | 이미 yield된 데이터 모델 항목 추적 |
| `_deleted_surfaces` | `Set[str]` | `set()` | 삭제된 서피스 ID 집합 |
| `_yielded_ids` | `Dict[str, Set[str]]` | `{}` | 서피스별 yield된 컴포넌트 ID 집합 |
| `_yielded_contents` | `Dict[Any, str]` | `{}` | `(surfaceId, cid)` → JSON 직렬화 문자열 (변경 감지용) |
| `_root_ids` | `Dict[str, str]` | `{}` | 서피스 ID별 루트 컴포넌트 ID 매핑 |
| `_default_root_id` | `Optional[str]` | `None` | 프로토콜 기본 루트 ID |
| `_unbound_root_id` | `Optional[str]` | `None` | surfaceId 확정 전 임시 루트 ID 보관 |
| `_surface_id` | `Optional[str]` | `None` | 현재 활성 서피스 ID |
| `_msg_types` | `List[str]` | `[]` | 현재 블록에서 발견된 메시지 타입 목록 |
| `_yielded_start_messages` | `Set[str]` | `set()` | beginRendering/createSurface를 이미 emit한 서피스 ID 집합 |
| `_active_msg_type` | `Optional[str]` | `None` | 컴포넌트 그룹화에 사용되는 현재 활성 메시지 타입 |
| `_pending_messages` | `Dict[str, List[Dict]]` | `{}` | start 메시지 수신 전까지 지연된 메시지 (surfaceId → list) |
| `_buffered_start_message` | `Optional[Dict]` | `None` | 컴포넌트보다 먼저 yield할 start 메시지 |
| `_topology_dirty` | `bool` | `False` | 컴포넌트가 비순서적으로 추가된 경우 `True` |
| `_found_valid_json_in_block` | `bool` | `False` | 현재 블록에서 유효한 JSON 객체를 파싱했는지 여부 |

---

#### 프로퍼티

**`_placeholder_component -> Dict[str, Any]`** (추상)
버전별 로딩 플레이스홀더 컴포넌트 딕셔너리를 반환한다. 서브클래스 구현 필수. 미구현 시 `NotImplementedError`.

**`surface_id -> Optional[str]`** (getter/setter)
현재 활성 서피스 ID를 읽거나 설정한다. setter에서 값이 `None`이 아닐 때, `_unbound_root_id`가 있으면 `_root_ids[value]`에 바인딩하고 `_unbound_root_id`를 `None`으로 초기화한다.

**`root_id -> Optional[str]`** (getter/setter)
현재 서피스의 루트 컴포넌트 ID를 반환한다. `_surface_id`가 있으면 `_root_ids.get(_surface_id, _default_root_id)`, 없으면 `_unbound_root_id` 또는 `_default_root_id`를 반환한다. setter는 `_surface_id` 유무에 따라 `_root_ids` 또는 `_unbound_root_id`에 값을 저장한다. `_surface_id`가 있고 값이 `None`이면 `_root_ids`에서 해당 키를 삭제한다.

**`msg_types -> List[str]`** (getter)
현재 블록에서 누적된 메시지 타입 목록을 반환한다.

**`_yielded_surfaces_set -> Set[str]`** (추상)
버전별 yield된 서피스 집합을 반환한다. 서브클래스 구현 필수.

**`_data_model_msg_type -> str`** (추상)
데이터 모델 업데이트 메시지 타입 문자열을 반환한다. 서브클래스 구현 필수.

---

#### `add_msg_type(self, msg_type: str)`

`_msg_types`에 `msg_type`이 없으면 추가한다. `msg_type`이 `MSG_TYPE_SURFACE_UPDATE`, `MSG_TYPE_UPDATE_COMPONENTS`, `MSG_TYPE_CREATE_SURFACE` 중 하나이면 `_active_msg_type`도 해당 값으로 갱신한다.

---

#### 추상 메서드 (서브클래스 구현 필수)

| 메서드 | 목적 |
|--------|------|
| `is_protocol_msg(self, obj)` | 객체가 이 버전의 A2UI 프로토콜 메시지인지 확인 |
| `_get_active_msg_type_for_components(self)` | 컴포넌트 래핑 시 사용할 msg_type 반환 |
| `_handle_complete_object(self, obj, sid, messages)` | 완전히 파싱된 최상위 객체 처리 |
| `_sniff_metadata(self)` | 버퍼에서 surfaceId, root, msg_types를 탐지 |

---

#### `_deduplicate_data_model(self, m: Dict, strict_integrity: bool) -> bool`

기본 구현은 항상 `True`를 반환한다 (서브클래스에서 오버라이드 가능). 데이터 모델 메시지 중복 yield를 막는 훅이다.

---

#### `_yield_messages(self, messages_to_yield: List[Dict], messages: List[ResponsePart], strict_integrity: bool = True)`

**목적:** 메시지 리스트를 검증 후 `messages` 출력 리스트에 추가한다.

**동작 단계:**
1. 각 메시지 `m`에 대해 `_deduplicate_data_model`로 중복 여부를 확인한다. `False`이면 건너뛴다.
2. `_validator`가 있으면 `_validator.validate(m, root_id=self.root_id, strict_integrity=...)`를 호출한다.
   - `strict_integrity=True`일 때 검증 실패는 `ValueError`로 전파된다.
   - `strict_integrity=False`일 때 검증 실패는 `logger.debug` 기록 후 해당 메시지를 건너뛴다.
3. `messages`의 마지막 항목이 `a2ui_json=None`이면 그 항목의 `a2ui_json`을 `[m]`으로 설정한다. 마지막 항목이 이미 리스트이면 `append`한다. 그 외에는 새 `ResponsePart(a2ui_json=[m])`을 추가한다.

---

#### `_delete_surface(self, sid: str)`

지정된 서피스 ID에 관련된 모든 상태를 제거한다. `_pending_messages`, `_yielded_ids`에서 `sid` 키를 삭제하고, `_yielded_contents`에서 `k[0] == sid`인 항목을 모두 제거한다. `_yielded_surfaces_set`과 `_yielded_start_messages`에서 `sid`를 제거하고, `_deleted_surfaces`에 `sid`를 추가한다.

---

#### `process_chunk(self, chunk: str) -> List[ResponsePart]`

**목적:** 스트리밍 파서의 주 진입점. 원시 텍스트 청크를 받아 완성된 A2UI 메시지를 즉시 반환한다.

**동작 단계:**
1. `chunk`를 `_buffer`에 누적한다.
2. 외부 `while True` 루프:
   - `_found_delimiter`가 `False`인 경우 (오픈 태그 탐색 단계):
     - `A2UI_OPEN_TAG`가 버퍼에 있으면 태그 앞 텍스트를 `ResponsePart(text=...)`로 emit하고 `_found_delimiter = True`로 설정, 버퍼를 태그 뒤로 갱신한다.
     - 없으면 태그의 접두사가 버퍼 끝에 걸릴 수 있으므로, 안전하게 yield 가능한 길이를 계산하여 앞부분 텍스트를 emit하고 나머지를 버퍼에 보관한 뒤 `break`한다.
   - `_found_delimiter`가 `True`인 경우 (클로즈 태그 탐색 단계):
     - `A2UI_CLOSE_TAG`가 버퍼에 있으면 태그 앞 JSON 단편을 `_process_json_chunk`에 전달한다. `_found_valid_json_in_block`이 `False`이면 `ValueError`를 raise한다. `_found_delimiter = False`로 재설정, `_reset_json_state()` 호출, 버퍼를 태그 뒤로 갱신하고 루프 계속.
     - 없으면 클로즈 태그 접두사 보호를 위해 안전한 길이만큼 `_process_json_chunk`를 호출하고 `break`한다.
3. 루프 후 `messages`에서 같은 서피스의 중복 `surfaceUpdate` 메시지를 역방향 탐색으로 중복 제거한다 (가장 마지막, 즉 가장 완전한 메시지만 유지).
4. 결과 `messages` 리스트를 반환한다.

---

#### `_reset_json_state(self)`

블록 종료 시 JSON 파싱 상태를 리셋한다. `_json_buffer`, `_brace_stack`, `_brace_count`, `_in_top_level_list`, `_in_string`, `_string_escaped`, `_msg_types`, `_found_valid_json_in_block`을 초기값으로 리셋한다. `_active_msg_type`과 `_yielded_contents`는 리셋하지 않는다 (블록 간 재yield 허용).

---

#### `_fix_json(self, fragment: str) -> str`

**목적:** 불완전한 JSON 단편에 닫는 구분자를 추가하여 파싱 가능한 형태로 복구한다.

**동작 단계:**
1. 단일 패스로 문자열을 순회하며 이스케이프 상태, 문자열 내부 여부, 여는 괄호/대괄호 스택을 추적한다.
2. 문자열이 열린 채로 끝난 경우(`in_string=True`):
   - 키 자리에 있는 경우(`prefix.endswith(":")`) `_cuttable_keys`에 포함되지 않는 키면 빈 문자열을 반환 (yield 불가).
   - `valueString` 키이고 값이 `http://`, `https://`, `data:`, `/`로 시작하는 URL/경로 형태이거나, 데이터 모델 key에 `url`, `link`, `src`, `href`, `image` 중 하나가 포함된 경우 빈 문자열 반환.
   - 그 외에는 닫는 `"` 추가.
3. 후행 쉼표가 있으면 제거한다.
4. `stack`에 남은 여는 괄호/대괄호를 역순으로 닫는 구분자(`}` 또는 `]`)를 추가한다.
5. 복구된 문자열 반환. 입력이 비어 있으면 빈 문자열 반환.

---

#### `_process_json_chunk(self, chunk: str, messages: List[ResponsePart])`

**목적:** JSON 단편 청크를 문자 단위로 처리하여 완성된 JSON 객체를 즉시 파싱하고 컴포넌트/프로토콜 메시지를 처리한다.

**동작 상세:**
- 최상위 `[`를 발견할 때까지 다른 문자는 모두 건너뛴다 (`_in_top_level_list` 전환).
- 문자열 내부 상태(`_in_string`, `_string_escaped`)를 관리하여 문자열 내의 `{`, `}`, `[`, `]`를 잘못 카운팅하지 않는다.
- `{` 발견 시 `_brace_stack`에 `("{", 현재 json_buffer 길이)` push, `_brace_count` 증가.
- `}` 발견 시 스택 pop, `_brace_count` 감소. 파싱된 객체 버퍼(`obj_buffer`)로 `json.loads` 시도:
  - 성공 시 `_found_valid_json_in_block = True`.
  - `id`와 `component` 키가 있으면 `_handle_partial_component` 호출.
  - 최상위 객체이거나 프로토콜 메시지이면 `_handle_complete_object` 호출. 인식되지 않으면 `_yield_messages` 직접 호출.
  - 처리된 객체는 `_json_buffer`에서 제거하여 버퍼 비대화를 방지한다.
- `[`/`]` 처리도 마찬가지로 스택에 push/pop하며, 최상위 `]`를 닫으면 `_in_top_level_list = False`.
- 키 구분자(`"`, `:`, `,`, `}`, `]`) 발생마다 `_sniff_metadata()` 호출.
- 청크 끝에서 `_sniff_partial_component`, `_sniff_partial_data_model` 호출.
- `_topology_dirty`가 `True`이면 `yield_reachable(messages, check_root=False, raise_on_orphans=False)` 호출 후 `False`로 리셋.

---

#### `_construct_sniffed_data_model_message(self, active_msg_type: str, delta_msg_payload: Dict) -> Dict`

`{active_msg_type: delta_msg_payload}` 형태의 딕셔너리를 반환한다. 서브클래스에서 오버라이드 가능.

---

#### `_sniff_partial_data_model(self, messages: List[ResponsePart])`

**목적:** JSON 버퍼에서 미완성 데이터 모델 업데이트 메시지를 조기 탐지하여 delta를 yield한다.

**동작 단계:**
1. `_data_model_msg_type` 문자열이 버퍼에 없으면 즉시 반환한다.
2. `_brace_stack`을 역방향으로 순회하여 `{` 타입 항목마다 해당 위치의 JSON 단편을 `_fix_json`으로 복구하고 `json.loads`를 시도한다. 실패 시 후행 쉼표를 역방향으로 제거하며 재시도하는 fallback 루프를 사용한다.
3. 파싱 성공 객체에 `_data_model_msg_type` 키가 있으면 `contents` 필드를 `_parse_contents_to_dict`로 딕셔너리화한다.
4. 이미 yield된 `_yielded_data_model`과 비교하여 변경된 키만 `delta`로 추출한다. `delta`가 비어 있으면 skip.
5. `raw_contents`가 리스트이면 최신 항목만 역방향 중복 제거하여 `delta_contents`를 만들고, `_prune_incomplete_datamodel_entries`로 불완전 항목을 제거한다.
6. `delta_msg_payload`를 구성(`surfaceId`, `contents`, 선택적 `path`)하고 `_construct_sniffed_data_model_message`로 메시지를 만든다.
7. `_yield_messages([delta_msg], messages, strict_integrity=False)` 호출.
8. `_yielded_data_model` 업데이트 및 `update_data_model` 호출.

---

#### `_sniff_partial_component(self, messages: List[ResponsePart])`

**목적:** 버퍼에서 완전히 파싱된 컴포넌트 객체를 조기 탐지한다.

`CATALOG_COMPONENTS_KEY`가 버퍼에 없으면 즉시 반환한다. `_brace_stack`을 역방향 순회하여 `{` 항목마다 `_fix_json` 후 파싱을 시도한다. `id`와 `component` 키가 있고, `component`가 문자열(v0.9 flat)이거나 비어 있지 않은 딕셔너리(v0.8 nested)이면 `_handle_partial_component` 호출한다.

---

#### `_sniff_metadata(self)` (추상)

`_json_buffer`에서 `surfaceId`, 루트 ID, 메시지 타입을 탐지한다. 서브클래스 구현 필수.

---

#### `_prune_incomplete_datamodel_entries(self, entries: Any) -> Any`

**목적:** `key`만 있고 값 필드가 없는 불완전한 데이터 모델 항목을 재귀적으로 제거한다.

`entries`가 리스트가 아니면 그대로 반환한다. 각 항목에 대해 `value`, `valueString`, `valueNumber`, `valueBoolean` 중 하나라도 있으면 유효로 판단한다. `valueMap`이 있으면 재귀 호출하여 정리하고, 결과가 비어 있고 원본이 비어 있지 않았으면 `valueMap` 자체를 삭제한다. `key`가 있고 유효한 값이 있는 항목만 결과에 포함한다.

---

#### `_handle_partial_component(self, comp: Dict, messages: List[ResponsePart])`

**목적:** 부모 메시지 완성 전에 발견된 컴포넌트를 캐시에 저장하고 즉시 yield를 준비한다.

**동작 단계:**
1. `comp_id = comp.get("id")`가 없으면 즉시 반환.
2. 내부 헬퍼 `_has_empty_dict(obj)`를 정의하여 빈 딕셔너리(`{}`)를 재귀적으로 탐지한다. 딕셔너리가 비어 있으면 `True`, 값 중 하나라도 `_has_empty_dict`이면 `True`.
3. `component` 필드가 문자열(v0.9 flat style)이면 전체 `comp` 객체에서 빈 딕셔너리 검사. `component`가 딕셔너리(v0.8 nested style)이면 딕셔너리 내부만 검사. 빈 딕셔너리가 있으면 반환 (yield 불가).
4. v0.8 nested style에서 `_required_fields_map`이 있으면 컴포넌트 타입의 필수 필드 존재 여부를 확인한다. 필수 필드 중 하나라도 없으면 반환.
5. `_seen_components[comp_id] = comp`에 캐시, `_topology_dirty = True`로 설정.

---

#### `_parse_contents_to_dict(self, raw_contents: Any) -> Dict[str, Any]`

**목적:** A2UI `contents` 배열을 `{key: value}` 형태의 평면 딕셔너리로 변환한다.

`raw_contents`가 딕셔너리이면 그대로 반환한다. 리스트가 아니면 빈 딕셔너리 반환한다. 리스트인 경우 각 항목에서 `key`와 `value`/`valueString`/`valueNumber`/`valueBoolean` 중 첫 번째를 추출한다. `valueMap`이 있으면 재귀 호출하여 값으로 사용한다. `key`와 `val` 모두 존재할 때만 결과 딕셔너리에 포함한다.

---

#### `update_data_model(self, update: Dict, messages: List[ResponsePart])`

**목적:** 내부 데이터 모델을 갱신한다.

`update.get("contents")`가 있으면 `_parse_contents_to_dict`로 파싱하여 `contents` 딕셔너리를 얻는다. 없으면 `surfaceId`, `root`, `contents` 키를 제외한 나머지를 플랫 딕셔너리로 사용한다 (v0.8 flat structure fallback). 현재 기반 클래스 구현에서는 내용을 파싱하는 것 외에 추가 상태 업데이트는 없다 (서브클래스에서 확장 가능).

---

#### `yield_reachable(self, messages: List[Dict], check_root: bool = False, raise_on_orphans: bool = False)`

**목적:** 현재 `_seen_components`에서 루트로부터 도달 가능한 컴포넌트들을 부분 메시지로 즉시 yield한다. 스트리밍의 핵심 로직이다.

**동작 단계:**
1. `_get_active_msg_type_for_components()`를 호출하고, `root_id` 또는 결과가 없으면 즉시 반환.
2. `surface_id`가 없으면 반환. 서피스가 `_yielded_surfaces_set`에 없고 `_buffered_start_message`도 없으면 반환.
3. `analyze_topology(root_id, seen_components, ref_fields_map, raise_on_orphans=...)`로 도달 가능한 ID 집합을 구한다.
4. `available_reachable = reachable_ids & set(_seen_components.keys())`로 실제 존재하는 컴포넌트만 선별한다.
5. `sorted(available_reachable)` 순서로 각 컴포넌트를 `copy.deepcopy`하고 `_process_component_topology`로 플레이스홀더 처리 및 자식 정리를 수행한다. 추가 플레이스홀더를 `extra_components`에 수집한다.
6. 새로 추가된 컴포넌트가 있거나(`available_reachable - yielded_for_surface`), 기존 컴포넌트의 JSON 직렬화가 변경되었으면 `should_yield = True`.
7. `should_yield`이고 `_buffered_start_message`가 있는데 아직 emit되지 않은 서피스라면 start 메시지를 먼저 엄격 검증 모드로 yield한다.
8. `_construct_partial_message(processed_components, active_msg_type)`으로 부분 메시지를 구성하고 `_yield_messages([partial_msg], messages, strict_integrity=False)` 호출.
9. `_yielded_ids`, `_yielded_contents`를 갱신한다.
10. `ValueError`는 "Circular reference" 관련이면 재throw, 그 외 토폴로지 오류는 스트리밍 중 silent ignore한다. `check_root=True`이거나 Circular/Self-reference 관련 메시지면 `raise`.

---

#### `_get_placeholder_id(self, child_id: str) -> str`

미발견 자식 컴포넌트의 플레이스홀더 ID를 `f"loading_{child_id}"` 형태로 반환한다.

---

#### `_process_component_topology(self, comp: Dict, extra_components: List[Dict], inline_resolved: bool = False)`

**목적:** 컴포넌트 내부를 재귀 탐색하여 경로 플레이스홀더를 처리하고 미발견 자식 컴포넌트를 플레이스홀더로 교체한다.

**내부 재귀 함수 `traverse(obj, parent_key=None)`:**
- `obj`가 딕셔너리인 경우:
  1. **경로 플레이스홀더 처리:** `path` 키가 있고 `/`로 시작하는 문자열이면, v0.8에서 `componentId`가 없으면 객체를 비우고 path만 재설정한다. v0.8에서 `path`가 올바른 형식이 아니면 `/` 접두사를 추가한다.
  2. **자식 정리:** `children`, `explicitList`, `child`, `contentChild`, `entryPointChild`, `componentId` 필드에 대해 리스트이면 각 자식 ID를 확인하여 `_seen_components`에 없으면 `loading_{child_id}` 플레이스홀더로 교체하고 `extra_components`에 플레이스홀더 컴포넌트를 추가한다. 자식 리스트가 비어 있고 버퍼에 아직 해당 필드가 열려 있으면 `loading_children_{comp_id}` 플레이스홀더를 추가한다. 문자열 자식(단일 참조)도 마찬가지로 처리한다.
  3. 딕셔너리의 모든 값에 대해 재귀 `traverse` 호출.
- `obj`가 리스트인 경우: 각 항목에 대해 재귀 `traverse` 호출.

**진입점:** `comp.get("component")`가 딕셔너리이면 `traverse(comp.get("component", {}))`로 시작. 그 외 flat style은 `traverse(comp)` 전체에서 시작.

## 동작 흐름

1. 외부 호출자가 LLM 스트림에서 텍스트 청크를 받을 때마다 `process_chunk(chunk)`를 호출한다.
2. `process_chunk`는 `_buffer`에 누적하며 `A2UI_OPEN_TAG`를 찾을 때까지 텍스트를 emit하고, 태그를 발견하면 JSON 처리 모드로 전환한다.
3. JSON 처리 모드에서 `_process_json_chunk`가 문자 단위로 JSON을 파싱하며 완성된 객체를 즉시 처리한다. 각 완성 객체에 대해 프로토콜 메시지인지(`_handle_complete_object`) 아니면 컴포넌트인지(`_handle_partial_component`) 판단한다.
4. `_handle_partial_component`는 캐시에 저장하고 `_topology_dirty = True`로 표시, 청크 끝에 `yield_reachable`을 트리거한다.
5. `yield_reachable`은 루트로부터 도달 가능한 컴포넌트만 플레이스홀더 처리 후 부분 메시지로 즉시 emit한다.
6. `A2UI_CLOSE_TAG`를 발견하면 블록 상태를 리셋하고 다음 블록을 탐색한다.
7. 최종적으로 `process_chunk`는 `ResponsePart` 리스트를 반환하며, 호출자는 이를 UI 렌더러에 전달한다.
