# agent_sdks/python/a2ui_core/tests/test_validating.py

## 개요

`a2ui.core.validating` 모듈의 검증 로직 전체를 pytest로 테스트하는 파일이다. 컴포넌트 무결성 검사(`validate_component_integrity`), 위상 분석(`analyze_topology`), 경로·재귀 검증(`validate_recursion_and_paths`), 그리고 최상위 `A2uiValidator` 클래스의 프로토콜 봉투 검증 및 `ValidationConfig` 적용까지 다룬다. 소스 코드를 수정하지 않으며, 오직 기대 동작과 오류 메시지 패턴만 단언한다.

## 의존성

### 외부 패키지
- `typing` — `Any`, `Dict`, `List`, `Set`, `Tuple`, `Literal` (표준 라이브러리, 타입 힌트용)
- `pytest` — 테스트 실행 및 `pytest.raises` 컨텍스트 매니저
- `pydantic` — `BaseModel` (import만 존재, 직접 사용 없음)

### 저장소 내부 모듈
- [`a2ui.core.validating`](../a2ui/core/validating.py.md) — `A2uiValidator`, `A2uiValidatorError`, `analyze_topology`, `validate_component_integrity`, `validate_recursion_and_paths`, `get_component_references`, `ValidationConfig`, `CatalogValidator`
- [`a2ui.core.basic_catalog`](../a2ui/core/basic_catalog.py.md) — `BasicCatalog`

## Exports

테스트 파일이므로 공개 API를 내보내지 않는다. 모든 함수는 pytest가 수집·실행하는 테스트 케이스다.

## 테스트 케이스 명세

### 섹션 1 — 무결성 검사 (Integrity Checker Tests)

#### `test_get_component_references()`
- **검증 동작**: `get_component_references`가 단일 문자열 참조(`singleChild`), 리스트 참조(`childrenList`), 중첩 객체 참조(`nestedObj`의 `componentId` 키), 객체 리스트 내 참조(`tabs[*].child`) 등 모든 참조 유형을 올바르게 추출하는지 확인한다.
- **픽스처**: `ref_map`에 `"Container"` 컴포넌트에 대해 단일 참조 필드 집합 `{"singleChild", "nestedObj"}`와 리스트 참조 필드 집합 `{"childrenList", "tabs"}`를 정의한다. 컴포넌트 딕셔너리는 각각 `"child1"` (직접 문자열), `["child2", "child3"]` (리스트), `{"componentId": "child4"}` (중첩 객체), `[{"child": "tab1"}, {"child": "tab2"}]` (딕셔너리 리스트) 형태로 구성된다.
- **단언**: 반환된 참조 ID 목록에 `"child1"`, `"child2"`, `"child3"`, `"child4"`, `"tab1"`, `"tab2"` 여섯 개가 모두 존재.

#### `test_validate_component_integrity_valid()`
- **검증 동작**: 유효한 컴포넌트 목록(루트 포함, 중복 없음, 참조 대상 존재)이 예외 없이 통과하는지 확인한다.
- **픽스처**: `ref_map`에 `"Box"` 컴포넌트의 단일 참조 필드 `child` 정의. `"root"` 컴포넌트가 `"c1"`을 `child`로 참조하고, `"c1"`은 빈 `Box`로 단독 존재.
- **단언**: 예외 없이 완료.

#### `test_validate_component_integrity_duplicate_id()`
- **검증 동작**: 동일한 `id` 값 `"c1"`을 가진 두 컴포넌트가 있을 때 `ValueError`가 발생하는지 확인한다.
- **픽스처**: 컴포넌트 목록에 `id="c1"` 항목이 두 개. `ref_map`은 빈 딕셔너리.
- **단언**: `pytest.raises(ValueError, match="Duplicate component ID: c1")`.

#### `test_validate_component_integrity_missing_root()`
- **검증 동작**: `id="root"` 컴포넌트가 없을 때 `ValueError`가 발생하는지 확인한다.
- **픽스처**: `id="c1"` 컴포넌트 하나만 존재. `ref_map`은 빈 딕셔너리.
- **단언**: `match="Missing root component: No component has id='root'"`.

#### `test_validate_component_integrity_dangling_ref()`
- **검증 동작**: `root` 컴포넌트의 `child` 필드가 존재하지 않는 ID `"nonexistent"`를 참조할 때 `ValueError`가 발생하는지 확인한다.
- **픽스처**: `ref_map`에 `"Box"`의 단일 참조 필드 `child` 정의. 컴포넌트 목록에는 `"root"`만 존재.
- **단언**: `match="references non-existent component 'nonexistent'"`.

#### `test_validate_recursion_and_paths_valid()`
- **검증 동작**: 유효한 경로 문자열 `"/valid/path"`와 `"/another"`가 포함된 중첩 딕셔너리에서 예외 없이 통과하는지 확인한다.
- **픽스처**: `{"path": "/valid/path", "nested": [{"path": "/another"}]}`.
- **단언**: 예외 없이 완료.

#### `test_validate_recursion_and_paths_invalid_path()`
- **검증 동작**: 잘못된 경로 문자열 `"invalid~path//double"`이 포함된 딕셔너리가 `ValueError`를 유발하는지 확인한다.
- **단언**: `match="Invalid path syntax"`.

#### `test_validate_recursion_and_paths_global_depth()`
- **검증 동작**: 전역 재귀 깊이 한도(50)를 초과하는 52단계 중첩 리스트가 `ValueError`를 유발하는지 확인한다.
- **픽스처 생성**: `deep_list: Any = []`로 초기화한 뒤 루프 52회 동안 `deep_list = [deep_list]`로 감싸 최종 52단계 중첩 리스트를 만든다.
- **단언**: `match="Global recursion limit exceeded"`.

#### `test_validate_recursion_and_paths_func_depth()`
- **검증 동작**: `functionCall` 중첩 깊이 한도(5)를 초과하는 6단계 `call`/`args` 체인이 `ValueError`를 유발하는지 확인한다.
- **픽스처 생성**: `deep_call: Dict[str, Any] = {}`을 루트로, `curr = deep_call` 포인터를 유지하며 루프 6회 동안 `curr["call"] = "func"`, `curr["args"] = {}`, `curr = curr["args"]` 패턴으로 6단계 중첩을 구성한다.
- **단언**: `match="Recursion limit exceeded: functionCall depth"`.

---

### 섹션 2 — 위상 분석 (Topology Analyzer Tests)

#### `test_analyze_topology_valid()`
- **검증 동작**: root → n1로 연결된 유효한 그래프에서 `analyze_topology`가 방문한 노드 집합을 올바르게 반환하는지 확인한다.
- **픽스처**: `ref_map`에 `"Node"`의 단일 참조 필드 `next` 정의. `allow_orphan_components=False`.
- **단언**: 반환값 `== {"root", "n1"}`.

#### `test_analyze_topology_self_ref()`
- **검증 동작**: `root` 컴포넌트의 `next` 필드가 자기 자신 `"root"`를 가리킬 때 `ValueError`가 발생하는지 확인한다.
- **단언**: `match="Self-reference detected: Component 'root' references itself"`.

#### `test_analyze_topology_circular_ref()`
- **검증 동작**: root → n1 → root 순환 참조가 있을 때 `ValueError`가 발생하는지 확인한다.
- **단언**: `match="Circular reference detected involving component 'root'"`.

#### `test_analyze_topology_orphans()`
- **검증 동작**: root에서 도달할 수 없는 고아 컴포넌트 `"orphan"`이 있고 `allow_orphan_components=False`일 때 `ValueError`가 발생하는지 확인한다.
- **픽스처**: `"root"` 컴포넌트는 `next` 없이 빈 상태. `"orphan"` 컴포넌트는 독립적으로 존재.
- **단언**: `match="Component 'orphan' is not reachable from 'root'"`.

---

### 섹션 3 — 최상위 검증기 (Validator Tests)

#### `test_a2ui_validator_protocol_envelope_invalid_version()`
- **검증 동작**: `"version"` 필드가 없는 메시지 `[{"not_version": "v0.8"}]`를 `validate_protocol_envelope`에 전달했을 때 `A2uiValidatorError`가 발생하는지 확인한다.
- **단언**: `match="'version' is a required property|'v0.9' was expected|Field required"` (세 패턴 중 하나 이상).

#### `test_a2ui_validator_protocol_envelope_not_dict()`
- **검증 동작**: 메시지 원소가 딕셔너리가 아닌 문자열 `"not_a_dict"`일 때 `A2uiValidatorError`가 발생하는지 확인한다.
- **단언**: `match="Message must be an object"`.

#### `test_a2ui_validator_validate_valid_payload()`
- **검증 동작**: `BasicCatalog` 기반 `CatalogValidator`에 완전히 유효한 v0.9 메시지 두 개(`createSurface`, `updateComponents`)를 전달했을 때 예외 없이 통과하는지 확인한다.
- **픽스처**: `BasicCatalog()` 인스턴스, `A2uiValidator()` 인스턴스. `createSurface`는 `surfaceId="main"`, `catalogId="https://a2ui.org/catalog"`, `theme={"primaryColor": "#000000"}`. `updateComponents`는 `"Column"` 루트(`children: ["c1"]`)와 `"Text"` 자식(`text: "Hello"`).
- **단언**: 예외 없이 완료.

#### `test_a2ui_validator_validate_components_error()`
- **검증 동작**: 존재하지 않는 컴포넌트 타입 `"NonexistentComponent"`를 사용한 `updateComponents` 메시지가 `A2uiValidatorError`를 유발하는지 확인한다.
- **단언**: `pytest.raises(A2uiValidatorError)` (메시지 패턴 미지정).

#### `test_topology_cyclomatic_orphans_coverage()`
- **검증 동작**: 고아 컴포넌트, 순환 참조, 자기 참조 세 가지 위상 오류를 단일 테스트 함수에서 순서대로 검증해 분기 커버리지를 확보한다.
- **픽스처**: 모두 `ref_map = {"Node": ({"child"}, set())}` 공통 사용.
  1. `components_orphan`: root의 `child`가 `"A"`, `"A"`는 단독 존재, `"B"`도 단독 존재 → `"is not reachable from 'root'"` 오류 (`allow_orphan_components=False`).
  2. `components_cycle`: root → A → B → A 순환 → `"Circular reference detected"` 오류.
  3. `components_self`: root의 `child`가 자기 자신 `"root"` → `"Self-reference detected"` 오류.

#### `test_integrity_dangling_and_duplicate_pointers()`
- **검증 동작**: 중복 ID와 댕글링 참조 두 가지 무결성 오류를 단일 테스트 함수에서 순서대로 검증한다.
- **픽스처**: `ref_map = {"Node": ({"child"}, set())}`.
  1. `components_dup`: `id="root"` 두 개 → `"Duplicate component ID"` 오류.
  2. `components_dangle`: root의 `child`가 `"MissingNode"` → `"references non-existent component"` 오류.

#### `test_validate_recursion_and_paths_syntax_coverage()`
- **검증 동작**: 실제 v0.9 페이로드 형태로 경로 문법 오류와 함수 호출 깊이 초과 두 가지 오류를 검증한다.
  1. `updateDataModel` 페이로드에서 `path` 값 `"/users/~3name"` (이스케이프되지 않은 포인터) → `match="Invalid path syntax"`.
  2. `updateComponents` 페이로드에서 `"text"` 필드에 6단계 중첩 함수 호출 (`nestedFunctionA` args → `nestedFunctionB` args → ... → `nestedFunctionF`) → `match="Recursion limit exceeded"`.

#### `test_validator_aggregated_pydantic_error_formatting()`
- **검증 동작**: `[{"version": "v0.9"}]` (메시지 타입 필드 없음)을 `validate_protocol_envelope`에 전달했을 때 발생하는 `A2uiValidatorError`의 문자열 표현에 집계된 Pydantic 오류 경로 표기 `"messages.0"`이 포함되는지 확인한다.
- **단언**: `"messages.0" in str(exc_info.value)`.

#### `test_validator_config_parameter()`
- **검증 동작**: `ValidationConfig`의 `allow_orphan_components`와 `allow_dangling_references` 플래그가 `validate_components` 및 `validate`에서 실제로 적용되는지 검증한다.
- **픽스처**:
  - `catalog = CatalogValidator.from_catalog(BasicCatalog())`
  - `strict_config = ValidationConfig(allow_orphan_components=False, allow_dangling_references=False)`
  - `relaxed_config = ValidationConfig(allow_orphan_components=True, allow_dangling_references=True)`
- **시나리오**:
  1. `orphan_components` = `[{"id": "root", "component": "Column", "children": []}, {"id": "orphan", "component": "Text", "text": "I am an orphan"}]`: strict → `"is not reachable from"` 오류, relaxed → 통과.
  2. `dangling_components` = `[{"id": "root", "component": "Column", "children": ["non_existent_id"]}]`: strict → `"references non-existent component"` 오류, relaxed → 통과.
  3. `dangling_components`를 담은 `updateComponents` v0.9 페이로드를 `validator.validate`로 검증: strict → `"references non-existent component"` 오류, relaxed → 통과.

## 동작 흐름

파일은 세 논리 섹션으로 구분된다. 섹션 1은 `get_component_references`, `validate_component_integrity`, `validate_recursion_and_paths` 등 저수준 헬퍼 함수를 직접 호출해 정상 경로와 각 오류 조건을 독립적으로 검증한다. 섹션 2는 `analyze_topology`의 그래프 순회 로직(정상, 자기 참조, 순환, 고아)을 격리 검증한다. 섹션 3은 `A2uiValidator` 최상위 API를 통해 실제 v0.9 프로토콜 페이로드를 대상으로 통합 검증하며, `ValidationConfig`가 완화 모드에서 오류를 억제하는 동작까지 확인한다. 테스트 간 공유 픽스처나 setup/teardown은 없으며, 각 테스트 함수가 필요한 데이터를 내부에서 직접 구성한다.
