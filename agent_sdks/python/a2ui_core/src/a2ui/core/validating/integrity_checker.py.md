# agent_sdks/python/a2ui_core/src/a2ui/core/validating/integrity_checker.py

## 개요

컴포넌트 목록의 무결성(중복 ID, 댕글링 참조, 루트 컴포넌트 존재)과 데이터 구조의 재귀 깊이·경로 구문을 검증하는 함수들을 제공한다. `validate_component_integrity`는 컴포넌트 ID 집합을 구성하고 참조 유효성을 확인하며, `validate_recursion_and_paths`는 중첩 깊이와 `path` 필드 구문을 검사한다. `get_component_references`는 컴포넌트가 참조하는 다른 컴포넌트 ID와 필드 경로를 열거하는 이터레이터를 반환하며, `topology_analyzer.py`에서도 공유 사용된다.

## 의존성

### 외부 패키지
- `typing`: `Any`, `Dict`, `List`, `Optional`, `Set`, `Tuple`, `Union`, `Iterator`
- `re`: 정규식 처리

### 저장소 내부 모듈
- [`../schema/constants.py`](../schema/constants.py.md) — `ROOT_ID`

## Exports

| 이름 | 종류 |
|---|---|
| `get_component_references` | 함수 |
| `validate_component_integrity` | 함수 |
| `validate_recursion_and_paths` | 함수 |
| `NUMERIC_PATTERN` | 상수 (컴파일된 정규식) |
| `MAX_GLOBAL_DEPTH` | 상수 (정수) |
| `MAX_FUNC_CALL_DEPTH` | 상수 (정수) |
| `RELAXED_PATH_PATTERN` | 상수 (컴파일된 정규식) |

## 상세 명세

### 모듈 수준 상수

- `NUMERIC_PATTERN`: `re.compile(r"^(?:0|[1-9][0-9]*)$")` — 문자열이 비음수 정수(0 또는 선행 0 없는 양의 정수)인지 판별하는 정규식
- `MAX_GLOBAL_DEPTH`: `50` — 전체(global) 재귀 깊이 상한
- `MAX_FUNC_CALL_DEPTH`: `5` — 함수 호출 중첩 깊이 상한
- `RELAXED_PATH_PATTERN`: `re.compile(r"^(?:(?:\/(?:[^~\/]|~[01])*)*|(?:[^~\/]|~[01])+(?:\/(?:[^~\/]|~[01])*)*)$")` — JSON Pointer 경로 구문의 완화된 패턴. `~0`(리터럴 `~`), `~1`(리터럴 `/`) 이스케이프를 허용하며 슬래시로 시작하는 절대 경로와 그렇지 않은 상대 경로 모두 허용한다.

---

### `get_component_references(component, ref_fields_map) -> Iterator[Tuple[str, str]]`

**시그니처**: `get_component_references(component: Dict[str, Any], ref_fields_map: Dict[str, Tuple[Set[str], Set[str]]]) -> Iterator[Tuple[str, str]]`

단일 컴포넌트가 참조하는 다른 컴포넌트 ID와 해당 필드 경로의 쌍을 이터레이터로 생성한다.

동작:
1. `component.get("component")`의 값을 확인한다.
2. 값이 문자열(`str`)이면, 해당 문자열이 `comp_type`이고 `component` 자체가 `props`다. `_get_refs_recursively(comp_type, component, ref_fields_map)`를 `yield from`한다.
3. 값이 비어 있지 않은 딕셔너리(`dict`)이면, `next(iter(comp_val.keys()))`로 첫 번째 키를 `comp_type`으로, `comp_val[comp_type]`을 `props`로 삼아 `_get_refs_recursively`를 `yield from`한다.
4. 위 두 경우가 아니면(예: `None`, 빈 dict 등) 아무것도 생성하지 않는다.

---

### `_get_refs_recursively(comp_type, props, ref_fields_map) -> Iterator[Tuple[str, str]]` (비공개)

**시그니처**: `_get_refs_recursively(comp_type: str, props: Dict[str, Any], ref_fields_map: Dict[str, Tuple[Set[str], Set[str]]]) -> Iterator[Tuple[str, str]]`

주어진 컴포넌트 타입의 참조 필드에서 `(참조 ID, 경로 문자열)` 쌍을 재귀 탐색으로 생성한다.

동작:
1. `comp_type`이 빈 문자열이거나 `props`가 딕셔너리가 아니면 즉시 반환(생성 없음).
2. `ref_fields_map.get(comp_type, (set(), set()))`으로 `(single_refs, list_refs)` 집합 쌍을 얻는다.
3. 내부 헬퍼 함수 `extract_pointers(val: Any, current_path: str) -> Iterator[Tuple[str, str]]`를 정의한다:
   - `val`이 `str`이면 `(val, current_path)` 쌍을 `yield`한다.
   - `val`이 `list`이면 `enumerate(val)`로 `(idx, item)`을 순회한다. `item`이 `str`이고 `"["` 문자가 `current_path`에 없으면 `sub_path = current_path`(경로 유지), 그 외에는 `sub_path = f"{current_path}[{idx}]"`. 각 `item`에 대해 `extract_pointers(item, sub_path)`를 `yield from`한다.
   - `val`이 `dict`이면 각 `(sub_key, sub_val)` 쌍에 대해 `extract_pointers(sub_val, f"{current_path}.{sub_key}")`를 `yield from`한다.
4. `props`의 모든 `(key, value)` 쌍을 순회하며 `key`가 `single_refs` 또는 `list_refs`에 있으면 `extract_pointers(value, key)`를 `yield from`한다.

---

### `validate_component_integrity(components, ref_fields_map, root_id, allow_dangling_references) -> None`

**시그니처**: `validate_component_integrity(components: List[Dict[str, Any]], ref_fields_map: Dict[str, Tuple[Set[str], Set[str]]], root_id: Optional[str] = ROOT_ID, allow_dangling_references: bool = False) -> None`

컴포넌트 목록의 구조적 무결성을 세 단계로 검증한다. `root_id`의 기본값은 `constants.py`의 `ROOT_ID`(`"root"`)이다.

동작:
1. **ID 수집 및 중복 검사**: `components`를 순회하며 `comp.get("id")`가 `None`이 아닌 항목을 처리한다. `comp_id`가 이미 `ids` 집합에 있으면 `ValueError(f"Duplicate component ID: {comp_id}")`를 발생시킨다. 없으면 `ids.add(comp_id)`를 수행한다.
2. **증분 업데이트 조기 반환**: `allow_dangling_references`가 `True`이면 이후 검사를 모두 건너뛰고 반환한다. 클라이언트에 이미 존재하는 컴포넌트를 참조하는 부분 업데이트를 허용하기 위한 경로다.
3. **루트 컴포넌트 확인**: `root_id`가 `None`이 아니고 `root_id`가 `ids`에 없으면 `ValueError(f"Missing root component: No component has id='{root_id}'")`를 발생시킨다.
4. **댕글링 참조 검사**: `components`를 다시 순회하며 각 `comp`에 대해 `comp.get("id", "Unknown")`을 `comp_id`로 사용하고, `get_component_references(comp, ref_fields_map)`로 `(ref_id, field_name)` 쌍을 순회한다. `ref_id`가 `ids`에 없으면 `ValueError(f"Component '{comp_id}' references non-existent component '{ref_id}' in field '{field_name}'")`를 발생시킨다.

---

### `validate_recursion_and_paths(data: Any) -> None`

중첩 데이터 구조의 전체 깊이와 함수 호출 중첩 깊이를 검사하고, `path` 필드의 구문을 검증한다. 내부 재귀 함수 `traverse(item, global_depth, func_depth)`를 정의하여 `traverse(data, 0, 0)`으로 시작한다.

`traverse(item: Any, global_depth: int, func_depth: int)` 로직:
1. `global_depth > MAX_GLOBAL_DEPTH`(50)이면 `ValueError(f"Global recursion limit exceeded: Depth > {MAX_GLOBAL_DEPTH}")`를 발생시킨다.
2. `item`이 `list`이면 각 원소 `x`에 대해 `traverse(x, global_depth + 1, func_depth)`를 호출하고 반환한다.
3. `item`이 `dict`이면:
   - `"path"` 키가 있고 값이 `str`이면 `re.fullmatch(RELAXED_PATH_PATTERN, path)`로 경로 구문을 검증한다. 불일치하면 `ValueError(f"Invalid path syntax: '{path}'")`를 발생시킨다.
   - `is_func_v08`: `"functionCall"` 키가 있고 `item["functionCall"]`이 `dict`이면 `True`.
   - `is_func_v09`: `"call"` 키와 `"args"` 키가 모두 있으면 `True`.
   - `is_func_v08`이 `True`이면: `func_depth >= MAX_FUNC_CALL_DEPTH`(5)이면 `ValueError(f"Recursion limit exceeded: functionCall depth > {MAX_FUNC_CALL_DEPTH}")`를 발생시킨다. 그렇지 않으면 `traverse(item["functionCall"], global_depth + 1, func_depth + 1)`을 호출한다.
   - `is_func_v09`이 `True`이면 (v0.8이 아닌 경우에만 진입): 동일하게 깊이 초과 시 `ValueError`를 발생시킨다. 초과하지 않으면 딕셔너리의 모든 `(k, v)` 쌍에 대해, `k == "args"`이면 `traverse(v, global_depth + 1, func_depth + 1)`을, 그 외이면 `traverse(v, global_depth + 1, func_depth)`를 호출한다.
   - 함수 호출이 아닌 일반 딕셔너리이면 모든 값 `v`에 대해 `traverse(v, global_depth + 1, func_depth)`를 호출한다.

## 동작 흐름

이 파일의 모든 함수는 입력 상태만 읽고 상태를 변경하지 않는 순수 함수다. `get_component_references`가 참조 추출의 핵심이며 `validate_component_integrity`와 `topology_analyzer.py`의 `analyze_topology` 양쪽에서 공유된다. `validate_component_integrity`는 참조 대상 ID의 존재 여부만 확인하며 그래프 순환을 탐지하지 않는다(순환 탐지는 `topology_analyzer.py`의 `analyze_topology` 담당). `validate_recursion_and_paths`는 스키마 검증과 독립적으로 원시 데이터 구조의 안전한 깊이와 경로 구문을 보장한다.
