# agent_sdks/python/a2ui_core/src/a2ui/core/catalog/json_catalog.py

## 개요

미리 컴파일된 Pydantic 모델 없이 원시 JSON Schema 딕셔너리만으로 카탈로그를 표현하는 `JsonCatalog` 클래스를 제공한다. 서버 측 추론 프롬프트 빌딩과 동적 검증에 특화되어 있으며, `CatalogApi`의 구체 구현체다. 카탈로그 JSON의 컴포넌트 속성을 파싱해 컴포넌트 간 참조 필드를 위상 정렬용 맵으로 추출하는 기능(`extract_ref_fields`)이 핵심이다.

## 의존성

### 외부 패키지
- `typing` — `Any`, `Dict`, `List`, `Optional`, `Set`, `Tuple`

### 저장소 내부 모듈
- [`../schema/constants.py`](../schema/constants.py.md) — `CATALOG_COMPONENTS_KEY`
- [`./catalog.py`](catalog.py.md) — `CatalogApi`

## Exports

| 이름 | 종류 |
|---|---|
| `JsonCatalog` | 클래스 |

## 상세 명세

### `JsonCatalog(CatalogApi)`

#### `__init__(self, spec_version: str, catalog_schema: Dict[str, Any], catalog_id: Optional[str] = None, common_types_schema: Optional[Dict[str, Any]] = None)`

1. `catalog_id`가 없으면 `catalog_schema.get("catalogId")`로 대체 시도.
2. 그래도 없으면 `ValueError("catalog_id must be provided or exist in catalog_schema.")` 발생.
3. 부모 `CatalogApi.__init__(spec_version, catalog_id)` 호출 — 여기서 추가 검증 수행.
4. `self.catalog_schema`와 `self.common_types_schema`를 저장.

#### `get_component_schema(self, comp_type: str) -> Optional[Dict[str, Any]]`
`catalog_schema["components"]` 딕셔너리에서 `comp_type` 키를 조회해 반환. 없으면 `None`.

#### `get_function_schema(self, func_name: str) -> Optional[Dict[str, Any]]`
`catalog_schema["functions"]` 딕셔너리에서 `func_name` 키를 조회해 반환. 없으면 `None`.

#### `get_theme_schema(self) -> Optional[Dict[str, Any]]`
`catalog_schema.get("theme")`를 반환.

#### `extract_ref_fields(self) -> Dict[str, Tuple[Set[str], Set[str]]]`

카탈로그 JSON에서 컴포넌트 간 참조 필드를 분석해 `{ 컴포넌트명: (단일_참조_집합, 리스트_참조_집합) }` 맵을 반환한다.

**내부 헬퍼 함수:**

**`is_component_id_ref(prop_schema: Dict[str, Any]) -> bool`**
- `prop_schema`가 dict가 아니면 `False`.
- `$ref` 값이 `"$defs/ComponentId"`로 끝나면 `True`.
- `oneOf`, `anyOf`, `allOf` 중 어느 배열 항목이라도 재귀적으로 `True`이면 `True`.

**`is_child_list_ref(prop_schema: Dict[str, Any]) -> bool`**
- `is_component_id_ref`와 동일 구조, 단 `$ref` 끝부분이 `"$defs/ChildList"`인지 검사.

**`resolve_ref(schema: Any, visited: Optional[Set[str]] = None) -> Any`**
- `schema`가 dict가 아니거나 `"$ref"` 키가 없으면 그대로 반환.
- `ref`가 문자열이 아니거나, `"#/"` 시작이 아니거나, 이미 방문(`visited`)했거나, `"/ComponentId"` 또는 `"/ChildList"`로 끝나면 `schema` 반환(무한 루프 방지 및 참조 레이프 보호).
- 그 외: `ref`를 `"/"` 기준으로 분해 후 `catalog_schema` 내를 순회해 참조 대상을 찾고 재귀 호출.

**`extract_from_props(comp_schema: Dict[str, Any])`** (각 컴포넌트마다 호출되는 중첩 함수)
1. `comp_schema["properties"]`를 순회하며 각 속성을 `resolve_ref`로 역참조한다.
2. 역참조된 속성이 `is_component_id_ref`이면 `single_refs`에 추가.
3. `is_child_list_ref`이면 `list_refs`에 추가.
4. 그 외 `type == "array"`이고 `items`가 있으면: `items`를 역참조 후 다시 ComponentId/ChildList 검사. 배열 항목이 nested object이면 그 object의 properties도 재귀 검사.
5. `allOf`, `oneOf`, `anyOf` 키가 있으면 각 서브스키마에 대해 `extract_from_props`를 재귀 호출.

**최종 집계**: `single_refs`나 `list_refs` 중 하나라도 비어 있지 않으면 `ref_map[comp_name]`에 `(single_refs, list_refs)` 튜플을 저장.

## 동작 흐름

인스턴스화 시 카탈로그 JSON 전체를 메모리에 보관한다. 스키마 조회 메서드들은 단순 딕셔너리 조회로 O(1) 동작한다. `extract_ref_fields()`는 호출 시점에 전체 컴포넌트 스키마를 순회하며 `$ref` 포인터를 재귀적으로 해소하여 참조 맵을 빌드한다. 순환 참조 방지를 위해 `visited` 집합을 사용하며, `ComponentId`와 `ChildList`에 대한 참조는 의도적으로 역참조하지 않고 그대로 둔다.
