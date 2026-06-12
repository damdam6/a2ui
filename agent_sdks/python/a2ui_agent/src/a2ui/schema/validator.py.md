# agent_sdks/python/a2ui_agent/src/a2ui/schema/validator.py

## 개요

A2UI JSON 페이로드를 JSON 스키마 기준으로 검증하고, 컴포넌트 무결성·위상·재귀 깊이·경로 문법까지 종합적으로 검사하는 모듈이다. 버전 v0.8과 v0.9 이상의 두 가지 검증 경로를 지원하며, `A2uiValidator` 클래스와 그것을 보조하는 다수의 모듈 수준 함수로 구성된다. 카탈로그 스키마와 실제 페이로드 사이의 참조 정합성, 순환 참조, 고아 컴포넌트 등도 감지한다.

## 의존성

### 외부 패키지
- `copy` (표준 라이브러리)
- `logging` (표준 라이브러리)
- `re` (표준 라이브러리)
- `typing` — `TYPE_CHECKING`, `Any`, `Dict`, `List`, `Optional`, `Set`, `Tuple`, `Union`, `Iterator`
- `jsonschema.Draft202012Validator`
- `referencing` — `Registry`, `Resource` (함수 내에서 지연 임포트)
- `referencing.jsonschema.DRAFT202012` (함수 내에서 지연 임포트)

### 저장소 내부 모듈
- [`./utils`](./utils.py.md) — `wrap_as_json_array`
- [`./catalog`](./catalog.py.md) — `A2uiCatalog` (TYPE_CHECKING 전용, 런타임 순환 임포트 방지)
- [`./constants`](./constants.py.md) — `BASE_SCHEMA_URL`, `CATALOG_COMPONENTS_KEY`, `CATALOG_ID_KEY`, `CATALOG_STYLES_KEY`, `VERSION_0_8`, `VERSION_0_9`

## Exports

- `RELAXED_PATH_PATTERN` (상수, `re.Pattern`)
- `MAX_GLOBAL_DEPTH` (상수, `int`)
- `MAX_FUNC_CALL_DEPTH` (상수, `int`)
- `COMPONENTS`, `ID`, `ROOT`, `PATH`, `FUNCTION_CALL`, `CALL`, `ARGS` (상수, `str`)
- `A2uiValidator` (클래스)
- `_find_root_id` (함수, 비공개 관례)
- `_validate_component_integrity` (함수, 비공개 관례)
- `analyze_topology` (함수)
- `extract_component_required_fields` (함수)
- `extract_component_ref_fields` (함수)
- `get_component_references` (함수)
- `get_refs_recursively` (함수)
- `_validate_recursion_and_paths` (함수, 비공개 관례)
- `_inject_additional_properties` (함수, 비공개 관례)

## 상세 명세

### 모듈 수준 상수

| 이름 | 값 | 설명 |
|---|---|---|
| `RELAXED_PATH_PATTERN` | 정규식 — RFC 6901 포인터와 상대 경로 모두 허용 | 경로 문법 검증에 사용 |
| `MAX_GLOBAL_DEPTH` | `50` | 전체 JSON 재귀 최대 깊이 |
| `MAX_FUNC_CALL_DEPTH` | `5` | `functionCall` 재귀 최대 깊이 |
| `COMPONENTS` | `"components"` | 컴포넌트 목록 키 |
| `ID` | `"id"` | 컴포넌트 ID 필드 키 |
| `ROOT` | `"root"` | 루트 컴포넌트 기본 ID 값 |
| `PATH` | `"path"` | 데이터 바인딩 경로 필드 키 |
| `FUNCTION_CALL` | `"functionCall"` | 오류 메시지에 사용 |
| `CALL` | `"call"` | FunctionCall 탐지 키 |
| `ARGS` | `"args"` | FunctionCall 탐지 키 |

---

### `_inject_additional_properties(schema, source_properties, mapping=None) -> Tuple[Dict, Set[str]]`

`schema`를 재귀적으로 순회하며, `additionalProperties: true`인 노드를 찾아 `source_properties`에서 동일한 키(`k`)를 가진 그룹을 주입하고 `additionalProperties: false`로 교체한다. 주입이 발생한 키 집합을 두 번째 반환값으로 돌려준다. 일치하는 그룹이 없는 노드는 그대로 두고 자식을 재귀한다.

내부에서 `recursive_inject(obj)`를 중첩 함수로 정의해 dict와 list를 모두 처리한다.

---

### `class A2uiValidator`

`A2uiCatalog`를 의존성으로 받아 JSON 스키마 검증기를 구성하고, A2UI 메시지 페이로드의 종합 검증을 수행하는 클래스.

#### `__init__(self, catalog: A2uiCatalog)`
`catalog`를 `_catalog`에 저장하고, `catalog.version`을 읽어 `version`에 설정(없으면 `VERSION_0_8` 기본값). `_build_validator()`를 호출해 `_validator`를 초기화한다.

#### `get_version(self) -> str`
`self.version`을 반환한다.

#### `_build_validator(self) -> Draft202012Validator`
`catalog.version`이 `VERSION_0_8`이면 `_build_0_8_validator()`를, 그 외에는 `_build_0_9_validator()`를 호출한다.

#### `_bundle_0_8_schemas(self) -> Dict[str, Any]`
`catalog.s2c_schema`가 없으면 빈 dict를 반환한다. `s2c_schema`를 deepcopy한 뒤, `catalog.catalog_schema`에서 `CATALOG_COMPONENTS_KEY`를 추출해 `source_properties["component"]`에 매핑하고(`"components"` → `"component"` 특수 변환), `CATALOG_STYLES_KEY`를 `source_properties["styles"]`에 매핑한다. `_inject_additional_properties`를 호출해 번들된 스키마를 반환한다.

#### `_build_0_8_validator(self) -> Draft202012Validator`
`_bundle_0_8_schemas()`로 번들 스키마를 구성하고 `wrap_as_json_array`로 감싼다. `s2c_schema.$id`를 기반으로 `common_types.json`의 절대 URI와 상대 URI를 모두 `Registry`에 등록한다. `$schema: "https://json-schema.org/draft/2020-12/schema"`를 validator_schema에 추가한 뒤 `Draft202012Validator`를 반환한다.

#### `_build_0_9_validator(self) -> Draft202012Validator`
`s2c_schema`를 `wrap_as_json_array`로 감싸고, `s2c_schema.$id`에서 `catalog.json`과 `common_types.json`의 절대 URI를 계산한다. `Registry`에 절대 URI 및 상대 URI 폴백(각 `"catalog.json"`, `"common_types.json"`)으로 네 개 이상의 리소스를 등록한다. `catalog.catalog_id`가 절대 URI와 다를 경우 추가로 등록한다. `Draft202012Validator`를 반환한다.

#### `validate(self, a2ui_json, root_id=None, strict_integrity=True) -> None`
입력이 list가 아니면 `[a2ui_json]`으로 감싼다. `version`이 `VERSION_0_9`이면 `_validate_0_9_custom`을 호출한다. 그 외(v0.8)는 `_validator.iter_errors`로 스키마 오류를 수집하고, 오류가 있으면 첫 번째 오류와 그 context를 포함한 `ValueError`를 발생시킨다. 스키마 오류가 없으면 각 메시지에서 `surfaceUpdate` 키를 찾아 `components`와 `surfaceId`를 추출하고, 컴포넌트가 있으면 `_validate_component_integrity`와 `analyze_topology`를 실행한다. 마지막으로 `_validate_recursion_and_paths`를 모든 메시지에 적용한다.

#### `_validate_0_9_custom(self, messages, root_id=None, strict_integrity=True) -> None`
각 메시지를 순회하며 메시지 타입 키(`"createSurface"`, `"updateComponents"`, `"updateDataModel"`, `"deleteSurface"`)를 식별한다. `"updateComponents"`는 `_get_update_components_errors`로, 나머지는 `_get_sub_validator`로 얻은 서브 검증기와 `_get_formatted_errors`로 처리한다. 알 수 없는 키는 오류 메시지로 추가한다. 모든 오류를 수집 후 한꺼번에 `ValueError`로 발생시킨다. 이후 컴포넌트 무결성 및 재귀/경로 검사를 v0.8과 동일한 방식으로 수행한다.

#### `_get_sub_validator(self, def_name: str) -> Draft202012Validator`
`s2c_schema.$defs`에서 `def_name` 정의를 찾는다. 없으면 `ValueError`. 해당 서브스키마와 기존 `_validator._registry`를 사용해 `Draft202012Validator`를 반환한다.

#### `_get_formatted_errors(self, validator, instance, base_path) -> List[str]`
`validator.iter_errors(instance)`를 실행해 오류 목록을 수집한다. 각 오류의 `path`를 점(`.`) 구분자로 결합해 `base_path`와 합친다. 오류 메시지에 `"Unevaluated properties are not allowed"` 또는 `"Additional properties are not allowed"`가 있고 괄호가 포함된 경우, 괄호 내부 내용만 추출한다. 형식화된 문자열 목록을 반환한다.

#### `_get_update_components_errors(self, message, path) -> List[str]`
`"version"` 키가 없거나 값이 `"v0.9"`가 아니면 오류를 추가한다. `"updateComponents"` 값이 dict가 아니면 즉시 반환한다. `"surfaceId"`가 없거나 str이 아니면 오류 추가. `"components"`가 list가 아니면 즉시 반환. 각 컴포넌트를 `_get_single_component_errors`로 검사한다.

#### `_get_single_component_errors(self, comp, path) -> List[str]`
`"component"` 필드를 읽어 카탈로그 스키마에서 해당 컴포넌트 정의를 찾는다. 없으면 오류 반환. 임시 스키마 `{"$schema": "...", "$ref": "catalog.json#/components/<comp_type>"}`를 생성해 `Draft202012Validator`를 빌드하고 `_get_formatted_errors`로 검사한다.

---

### `_find_root_id(messages, surface_id=None) -> Optional[str]`

메시지 목록을 순회하며 루트 ID를 찾는다. `"beginRendering"` 키가 있으면 `surfaceId`가 일치할 때 `beginRendering.root` 값을(없으면 `"root"`) 반환한다. `"createSurface"` 키가 있으면 `surfaceId`가 일치할 때 `"root"`를 반환한다. 둘 다 없으면 `None`을 반환한다.

---

### `_validate_component_integrity(root_id, components, ref_fields_map, skip_root_check=False) -> None`

세 가지를 순서대로 검사한다.
1. **중복 ID**: 컴포넌트 ID 집합(`ids`)을 구성하며 중복이 발견되면 `ValueError`.
2. **루트 컴포넌트**: `skip_root_check`가 False이고 `root_id`가 None이 아닌데 `ids`에 없으면 `ValueError`.
3. **댕글링 참조**: `root_id`가 있고 `skip_root_check`가 False인 경우에만, `get_component_references`로 각 컴포넌트의 참조 ID를 확인해 `ids`에 없으면 `ValueError`.

---

### `analyze_topology(root_id, components, ref_fields_map, raise_on_orphans=False) -> Set[str]`

인접 리스트(`adj_list`)를 구성하며 자기 참조가 발견되면 즉시 `ValueError`. 이후 DFS로 사이클과 깊이를 검사한다. 깊이가 `MAX_GLOBAL_DEPTH`를 초과하면 `ValueError`. 재귀 스택에 이미 있는 노드를 재방문하면 순환 참조로 `ValueError`.

`root_id`가 있으면 루트부터 DFS를 실행하고, `raise_on_orphans`가 True면 방문하지 않은 컴포넌트가 있을 때 `ValueError`. `root_id`가 없으면 모든 노드에 대해 DFS를 실행한다. 방문한 노드 집합을 반환한다.

---

### `extract_component_required_fields(catalog) -> Dict[str, Set[str]]`

버전에 따라 카탈로그 또는 s2c 스키마에서 컴포넌트 목록을 추출한다. 각 컴포넌트의 `"required"` 필드와 `allOf/oneOf/anyOf` 중첩 내의 `"required"` 필드를 재귀적으로 수집한다. `"component"` 자체는 제외한다. `{컴포넌트명: 필수_필드_집합}` dict를 반환한다.

---

### `extract_component_ref_fields(catalog) -> Dict[str, tuple[Set[str], Set[str]]]`

버전에 따라 컴포넌트 목록을 추출하고, 각 컴포넌트 속성이 단일 컴포넌트 참조인지(`is_component_id_ref`) 또는 자식 목록 참조인지(`is_child_list_ref`)를 판별한다.

**`is_component_id_ref`**: `$ref`가 `"ComponentId"`, `"child"`, `"/child"`로 끝나거나, `type: string + title: ComponentId` 패턴이면 True. `oneOf/anyOf/allOf`를 재귀 검사.

**`is_child_list_ref`**: `$ref`가 `"ChildList"`, `"children"`, `"/children"`로 끝나거나, `type: object`에 `explicitList/template/componentId` 속성이 있거나, `type: array`의 items가 컴포넌트 참조이면 True. `oneOf/anyOf/allOf` 재귀 검사.

`{컴포넌트명: (단일참조_필드_집합, 목록참조_필드_집합)}` dict를 반환한다.

---

### `get_component_references(component, ref_fields_map) -> Iterator[Tuple[str, str]]`

`"component"` 필드가 str이면 v0.9 플랫 형식으로 간주해 `get_refs_recursively`에 위임한다. `"component"` 필드가 dict이면 v0.8 중첩 형식으로 간주해 각 컴포넌트 타입별로 `get_refs_recursively`를 호출한다. `(참조_ID, 필드명)` 튜플을 yield한다.

---

### `get_refs_recursively(comp_type, props, ref_fields_map) -> Iterator[Tuple[str, str]]`

`ref_fields_map`에서 `comp_type`의 단일/목록 참조 필드를 가져온다. 명시적으로 매핑되지 않은 경우를 위해 휴리스틱 집합을 정의한다.
- `HEURISTIC_SINGLE`: `"child"`, `"contentChild"`, `"entryPointChild"`, `"detail"`, `"summary"`, `"root"`
- `HEURISTIC_LIST`: `"children"`, `"explicitList"`, `"template"`

각 속성 키를 순회하며:
- 단일 참조: 값이 str이면 `(값, 키)` yield; dict에 `"componentId"` 키가 있으면 `(componentId값, "key.componentId")` yield.
- 목록 참조: 값이 list이면 각 str 항목을 yield; dict이면 `"explicitList"`, `"template"`, `"componentId"` 패턴을 처리한다.
- 매핑되지 않은 list 속성에 대해서는 각 item dict의 `"child"` 키를 `"key[idx].child"` 형태로 yield한다 (탭 등 중첩 배열 특수 처리).

---

### `_validate_recursion_and_paths(data: Any) -> None`

내부에서 `traverse(item, global_depth, func_depth)` 중첩 함수를 정의해 data를 재귀 탐색한다.
- `global_depth > MAX_GLOBAL_DEPTH`이면 `ValueError`.
- list이면 각 항목에 `global_depth + 1`로 재귀.
- dict이면:
  - `"path"` 키가 있고 str이면 `RELAXED_PATH_PATTERN`으로 검증, 불일치 시 `ValueError`.
  - `"call"`과 `"args"` 키가 모두 있으면 `functionCall`로 간주: `func_depth >= MAX_FUNC_CALL_DEPTH`이면 `ValueError`; `"args"` 값에는 `func_depth + 1`로, 나머지에는 `func_depth`로 재귀.
  - 그 외 dict 값은 `func_depth` 유지한 채 재귀.

`traverse(data, 0, 0)`을 호출해 실행을 시작한다.

## 동작 흐름

`A2uiValidator`는 생성 시점에 카탈로그에서 버전을 읽어 적합한 `Draft202012Validator`를 한 번만 빌드한다. `validate()`가 호출되면 우선 JSON 스키마 검증을 수행하고, 통과한 경우에만 컴포넌트 무결성(`_validate_component_integrity`) → 위상 분석(`analyze_topology`) → 재귀·경로 검사(`_validate_recursion_and_paths`) 순으로 추가 검사를 실행한다. 모듈 수준 함수들(`extract_component_ref_fields`, `get_component_references` 등)은 `validate()` 내에서 그때마다 호출되어 카탈로그 스키마로부터 참조 정보를 동적으로 추출한다.
