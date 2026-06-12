# agent_sdks/python/a2ui_agent/src/a2ui/schema/catalog.py

## 개요

A2UI 컴포넌트 카탈로그의 설정(`CatalogConfig`)과 처리된 카탈로그 객체(`A2uiCatalog`)를 정의한다. `A2uiCatalog`는 JSON 스키마(server-to-client, common_types, catalog)를 보유하며 LLM 지시문 렌더링, 사용되지 않는 컴포넌트/메시지/타입 가지치기(pruning), 예시 파일 로드 등의 기능을 제공한다. `CatalogConfig`는 파일시스템 경로 또는 URI를 기반으로 카탈로그를 로드하기 위한 팩토리 역할을 한다.

## 의존성

### 외부 패키지
- `collections` — `deque` (BFS 순회)
- `copy` — `deepcopy` (스키마 복사)
- `glob` — 예시 파일 glob 패턴 검색
- `json` — JSON 직렬화
- `logging` — 경고 로그
- `os` — 파일시스템 접근
- `dataclasses` — `dataclass`, `field`, `replace`
- `typing` — `Any`, `Dict`, `List`, `Optional`, `Union`, `TYPE_CHECKING`
- `urllib.parse` — `urlparse`

### 저장소 내부 모듈
- [`./catalog_provider`](./catalog_provider.py.md) — `A2uiCatalogProvider`, `FileSystemCatalogProvider`
- [`./constants`](./constants.py.md) — `A2UI_SCHEMA_BLOCK_START`, `A2UI_SCHEMA_BLOCK_END`, `CATALOG_COMPONENTS_KEY`, `CATALOG_ID_KEY`, `DEFAULT_CUTTABLE_KEYS`, `VERSION_0_8`, `ENCODING`
- [`./validator`](./validator.py.md) — `A2uiValidator`

## Exports

- 데이터클래스: `CatalogConfig`
- 데이터클래스(frozen): `A2uiCatalog`
- 함수: `resolve_examples_path`
- 내부 헬퍼 함수: `_collect_refs`, `_prune_defs_by_reachability`

## 상세 명세

### 함수 `resolve_examples_path(path: Optional[str]) -> Optional[str]`

`path`가 `None`이면 `None`을 반환한다. 그렇지 않으면 `urlparse`로 파싱하여 스킴이 없거나 `"file"`이면 `parsed.path`를 반환한다. 다른 스킴이면 `ValueError`를 발생시킨다.

### 함수 `_collect_refs(obj: Any) -> set[str]`

JSON 객체 트리를 재귀적으로 순회하며 모든 `"$ref"` 문자열 값을 집합으로 수집한다. dict에서는 키가 `"$ref"`이고 값이 str인 경우 추가하고, 그 외 값은 재귀 처리한다. list는 각 항목을 재귀 처리한다.

### 함수 `_prune_defs_by_reachability(defs: Dict[str, Any], root_def_names: List[str], internal_ref_prefix: str = "#/$defs/") -> Dict[str, Any]`

BFS를 이용하여 `root_def_names`에서 도달 가능한 정의(`defs`)만 남기고 나머지를 제거한다.

매개변수:
- `defs: Dict[str, Any]` — 가지치기할 정의 딕셔너리
- `root_def_names: List[str]` — BFS 시작점 정의 이름 목록
- `internal_ref_prefix: str` — 내부 참조 식별 접두사 (기본: `"#/$defs/"`)

로직:
1. `visited_defs` 집합과 `refs_queue` 덱을 초기화.
2. 큐에서 이름을 꺼내, `defs`에 존재하고 아직 방문하지 않았으면 방문 표시.
3. 해당 정의 내의 모든 `$ref`를 수집하여 `internal_ref_prefix`로 시작하는 것을 큐에 추가.
4. 반환: `defs`에서 `visited_defs`에 속한 키만 포함한 딕셔너리.

### 데이터클래스 `CatalogConfig`

카탈로그 로드 설정을 담는 일반 dataclass이다.

필드:
- `name: str` — 카탈로그 이름
- `provider: A2uiCatalogProvider` — 스키마를 로드하는 프로바이더
- `examples_path: Optional[str] = None` — 예시 파일 경로 또는 glob 패턴
- `custom_cuttable_keys: Optional[frozenset[str]] = None` — 스트리밍 자동 닫기(heal) 허용 키 집합

#### 클래스 메서드 `from_path(cls, name: str, catalog_path: str, examples_path: Optional[str] = None, custom_cuttable_keys: Optional[frozenset[str]] = None) -> CatalogConfig`

로컬 경로 또는 `file://` URI로부터 `CatalogConfig`를 생성하는 팩토리.
- `urlparse(catalog_path)`로 스킴 판별.
- 스킴 없음 또는 `"file"`: `FileSystemCatalogProvider(parsed.path)` 생성.
- `"http"` 또는 `"https"`: `NotImplementedError` 발생.
- 그 외: `ValueError` 발생.
- `resolve_examples_path(examples_path)`로 예시 경로를 정규화하여 최종 인스턴스를 반환한다.

### 데이터클래스(frozen) `A2uiCatalog`

처리된 컴포넌트 카탈로그를 불변(immutable) 객체로 표현한다.

필드:
- `version: str` — 카탈로그 버전 문자열
- `name: str` — 카탈로그 이름
- `s2c_schema: Dict[str, Any]` — server-to-client JSON 스키마
- `common_types_schema: Dict[str, Any]` — 공통 타입 JSON 스키마
- `catalog_schema: Dict[str, Any]` — 컴포넌트 카탈로그 JSON 스키마
- `custom_cuttable_keys: Optional[frozenset[str]] = None` — 커스텀 cuttable 키

#### 프로퍼티 `cuttable_keys -> frozenset[str]`

`custom_cuttable_keys`가 설정되어 있으면 해당 frozenset을 반환하고, 없으면 `DEFAULT_CUTTABLE_KEYS`를 반환한다.

#### 프로퍼티 `catalog_id -> str`

`catalog_schema`에서 `CATALOG_ID_KEY`(`"catalogId"`)를 읽어 반환한다. 해당 키가 없으면 `ValueError` 발생.

#### 프로퍼티 `validator -> A2uiValidator`

`A2uiValidator(self)`를 생성하여 반환한다. 호출할 때마다 새 인스턴스를 생성한다.

#### 메서드 `_with_pruned_components(self, allowed_components: List[str]) -> A2uiCatalog`

`allowed_components`가 비어있으면 `self`를 그대로 반환한다. 그렇지 않으면:
1. `catalog_schema`를 깊은 복사.
2. `CATALOG_COMPONENTS_KEY` 아래의 컴포넌트 딕셔너리에서 `allowed_components`에 있는 것만 남긴다.
3. `$defs.anyComponent.oneOf` 목록에서 `#/components/<name>` 형식의 `$ref` 항목 중 허용된 이름만 남긴다. 알 수 없는 형식은 `logging.warning`으로 기록한다.
4. `replace(self, catalog_schema=schema_copy)`로 새 인스턴스를 반환한다.

#### 메서드 `_with_pruned_messages(self, allowed_messages: List[str]) -> A2uiCatalog`

`allowed_messages`가 비어있으면 `self`를 반환한다. 그렇지 않으면 버전에 따라 다르게 처리:
- `VERSION_0_8`: `s2c_schema["properties"]`를 `_prune_defs_by_reachability`로 가지치기 (참조 접두사: `"#/properties/"`).
- v0.9+: `s2c_schema["oneOf"]` 목록에서 허용된 메시지만 남기고, `s2c_schema["$defs"]`를 `_prune_defs_by_reachability`로 가지치기 (접두사: `"#/$defs/"`).
- `replace(self, s2c_schema=s2c_schema_copy)`로 새 인스턴스를 반환한다.

#### 메서드 `with_pruning(self, allowed_components: Optional[List[str]] = None, allowed_messages: Optional[List[str]] = None) -> A2uiCatalog`

공개 API로, `_with_pruned_components`, `_with_pruned_messages`, `_with_pruned_common_types`를 순서대로 적용하여 최종 가지치기된 카탈로그를 반환한다. 각 인수가 `None`이거나 빈 리스트이면 해당 단계를 건너뛴다.

#### 메서드 `_with_pruned_common_types(self) -> A2uiCatalog`

`common_types_schema`에 `$defs`가 없으면 `self`를 반환한다. 그렇지 않으면:
1. `catalog_schema`와 `s2c_schema`에서 모든 `$ref`를 수집한다.
2. `"common_types.json#/$defs/"`를 포함하는 ref에서 타입 이름을 추출하여 루트 집합을 구성한다.
3. `common_types_schema["$defs"]`를 `_prune_defs_by_reachability`로 가지치기한다.
4. `replace(self, common_types_schema=new_common_types_schema)`로 새 인스턴스를 반환한다.

#### 메서드 `render_as_llm_instructions(self) -> str`

카탈로그와 스키마를 LLM 시스템 프롬프트에 삽입 가능한 문자열로 렌더링한다.
1. `A2UI_SCHEMA_BLOCK_START`를 추가.
2. `s2c_schema`를 compact JSON(`separators=(",", ":")`)으로 변환하여 `"### Server To Client Schema:\n..."` 형식으로 추가.
3. `common_types_schema`에 비어있지 않은 `$defs`가 있으면 `"### Common Types Schema:\n..."` 형식으로 추가.
4. `catalog_schema`를 compact JSON으로 변환하여 `"### Catalog Schema:\n..."` 형식으로 추가.
5. `A2UI_SCHEMA_BLOCK_END`를 추가.
6. 모든 부분을 `"\n\n"`으로 결합하여 반환.

#### 메서드 `load_examples(self, path: Optional[str], validate: bool = False) -> str`

지정된 경로 또는 glob 패턴에서 예시 JSON 파일을 로드하여 하나의 문자열로 결합한다.
- `path`가 `None`이면 빈 문자열 반환.
- 경로가 디렉토리이면 `"/*.json"` 패턴을 자동 추가한다.
- `glob.glob(pattern, recursive=True)`로 파일 목록을 수집 후 정렬한다.
- 매칭 파일이 없으면 경로 형식에 따라 `logging.warning` 후 빈 문자열 반환.
- 각 파일을 `ENCODING`으로 읽어 `"---BEGIN {basename}---\n{content}\n---END {basename}---"` 형식으로 래핑한다.
- `validate=True`이면 각 파일에 대해 `_validate_example()` 호출.
- 최종 결과를 `"\n\n"`으로 결합하여 반환.

#### 메서드 `_validate_example(self, full_path: str, content: str) -> None`

`content`를 JSON으로 파싱하고 `self.validator.validate(json_data)`를 호출한다. 예외 발생 시 `ValueError`로 감싸 재발생시킨다.

## 동작 흐름

1. `CatalogConfig.from_path()`로 설정을 생성한다.
2. `A2uiSchemaManager`(또는 직접 코드)에서 `config.provider.load()`로 스키마를 로드한 뒤 `A2uiCatalog` 인스턴스를 구성한다.
3. 필요에 따라 `with_pruning()`으로 컴포넌트/메시지/공통 타입을 가지치기한다.
4. `render_as_llm_instructions()`로 LLM 프롬프트 삽입용 텍스트를 생성하거나, `load_examples()`로 예시를 로드한다.
5. `validator` 프로퍼티로 런타임 유효성 검사를 수행한다.
