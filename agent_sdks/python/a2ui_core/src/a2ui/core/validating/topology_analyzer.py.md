# agent_sdks/python/a2ui_core/src/a2ui/core/validating/topology_analyzer.py

## 개요

컴포넌트 목록으로 방향 그래프를 구성하고 DFS(깊이 우선 탐색)를 수행하여 자기 참조, 순환 참조, 고아(orphan) 컴포넌트, 최대 깊이 초과를 탐지하는 단일 함수 `analyze_topology`를 제공한다. 루트 컴포넌트를 시작점으로 삼아 도달 가능한 컴포넌트 집합을 반환하며, 루트가 없는 경우 모든 컴포넌트를 순회하여 사이클을 검사한다.

## 의존성

### 외부 패키지
- `typing`: `Any`, `Dict`, `List`, `Optional`, `Set`, `Tuple`

### 저장소 내부 모듈
- [`./integrity_checker.py`](integrity_checker.py.md) — `get_component_references`, `MAX_GLOBAL_DEPTH`
- [`../schema/constants.py`](../schema/constants.py.md) — `ROOT_ID`

## Exports

| 이름 | 종류 |
|---|---|
| `analyze_topology` | 함수 |

## 상세 명세

### `analyze_topology(components, ref_fields_map, root_id, allow_orphan_components) -> Set[str]`

**시그니처**:
```
analyze_topology(
    components: List[Dict[str, Any]],
    ref_fields_map: Dict[str, Tuple[Set[str], Set[str]]],
    root_id: Optional[str] = ROOT_ID,
    allow_orphan_components: bool = True,
) -> Set[str]
```

컴포넌트 의존성 그래프의 위상을 분석하여 방문한 컴포넌트 ID 집합을 반환한다. `root_id`의 기본값은 `constants.py`의 `ROOT_ID`(`"root"`)이며, `allow_orphan_components`의 기본값은 `True`임에 주의한다(상위 호출자 `A2uiValidator.validate_components`에서는 `config.allow_orphan_components`로 재정의할 수 있다).

**단계별 동작**:

1. **인접 리스트 구성**: `adj_list: Dict[str, List[str]] = {}`, `all_ids: Set[str] = set()`을 초기화한다. `components`를 순회하며 `comp.get("id")`가 `None`이 아닌 항목에 대해:
   - `all_ids.add(comp_id)`로 ID를 등록하고, `adj_list`에 `comp_id` 키가 없으면 빈 리스트로 초기화한다.
   - `get_component_references(comp, ref_fields_map)`로 `(ref_id, field_name)` 쌍을 순회한다:
     - `ref_id == comp_id`이면 자기 참조이므로 `ValueError(f"Self-reference detected: Component '{comp_id}' references itself in field '{field_name}'")`를 발생시킨다.
     - 그 외에는 `adj_list[comp_id].append(ref_id)`로 유향 간선을 추가한다.

2. **DFS 함수 정의**: `visited: Set[str] = set()`, `recursion_stack: Set[str] = set()`을 클로저 외부에서 초기화한다. 내부 재귀 함수 `dfs(node_id: str, depth: int)`를 정의한다:
   - `depth > MAX_GLOBAL_DEPTH`(50)이면 `ValueError(f"Global recursion limit exceeded: logical depth > {MAX_GLOBAL_DEPTH}")`를 발생시킨다.
   - `visited.add(node_id)`, `recursion_stack.add(node_id)`로 현재 노드를 표시한다.
   - `adj_list.get(node_id, [])`의 각 `neighbor`에 대해:
     - `neighbor`가 `visited`에 없으면 `dfs(neighbor, depth + 1)`을 재귀 호출한다.
     - `neighbor`가 `recursion_stack`에 있으면 사이클이 감지된 것이므로 `ValueError(f"Circular reference detected involving component '{neighbor}'")`를 발생시킨다.
   - 모든 이웃을 처리한 후 `recursion_stack.remove(node_id)`로 현재 노드를 재귀 스택에서 제거한다.

3. **탐색 실행**:
   - `root_id`가 `None`이 아닌 경우:
     - `root_id`가 `all_ids`에 있으면 `dfs(root_id, 0)`을 호출한다.
     - `allow_orphan_components`가 `False`이면 `orphans = all_ids - visited`를 계산한다. `orphans`가 비어 있지 않으면 `sorted(list(orphans))`의 첫 번째 원소를 사용하여 `ValueError(f"Component '{sorted_orphans[0]}' is not reachable from '{root_id}'")`를 발생시킨다.
   - `root_id`가 `None`인 경우 (부분 업데이트 등): `sorted(list(all_ids))`를 순회하며 아직 `visited`에 없는 `node_id`에 대해 `dfs(node_id, 0)`을 호출하여 전체 그래프의 사이클을 검사한다.

4. **반환**: `visited` 집합을 반환한다.

**경계 케이스**:
- `id`가 없는 컴포넌트는 그래프 노드로 등록되지 않는다.
- `root_id`가 `all_ids`에 없으면 DFS를 시작하지 않으므로 `visited`는 빈 집합이다. 이 상태에서 `allow_orphan_components=False`이면 모든 컴포넌트가 고아로 감지된다.
- `allow_orphan_components=True`(기본값)이면 루트에서 도달할 수 없는 컴포넌트가 있어도 오류를 발생시키지 않는다.

## 동작 흐름

`analyze_topology`는 순수 함수로 외부 상태를 변경하지 않는다. `get_component_references`(integrity_checker에서 임포트)를 이용해 각 컴포넌트의 이웃 노드를 수집하고, 표준 DFS + 재귀 스택 알고리즘으로 유향 그래프의 사이클을 탐지한다. 반환된 `visited` 집합은 루트에서 도달 가능한 모든 컴포넌트 ID를 나타내며, 상위 호출자(`A2uiValidator.validate_components`)에서 이 결과를 활용할 수 있다.
