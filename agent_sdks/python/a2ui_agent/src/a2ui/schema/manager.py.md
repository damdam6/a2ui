# agent_sdks/python/a2ui_agent/src/a2ui/schema/manager.py

## 개요

A2UI 스키마 수준 관리와 LLM 시스템 프롬프트 생성을 담당하는 `A2uiSchemaManager` 클래스를 제공한다. 지정된 버전의 server-to-client 스키마와 common types 스키마를 번들 리소스에서 로드하고, 하나 이상의 카탈로그 설정을 처리하여 클라이언트 UI 역량(capabilities)에 따른 적절한 카탈로그를 선택하며, 최종적으로 LLM에 전달할 시스템 프롬프트를 조립하는 역할을 한다.

## 의존성

### 외부 패키지
- `copy` — 카탈로그 스키마 깊은 복사
- `json` — JSON 처리
- `logging` — 로그
- `os` — 파일시스템 유틸
- `importlib.resources` — 번들 리소스 접근
- `typing` — `Any`, `Optional`, `Callable`
- `dataclasses` — `dataclass`, `field`

### 저장소 내부 모듈
- [`./utils`](./utils.py.md) — `load_from_bundled_resource`
- [`../inference_strategy`](../inference_strategy.py.md) — `InferenceStrategy` 기반 클래스
- [`./constants`](./constants.py.md) — 모든 스키마 상수 (와일드카드 import)
- [`./catalog`](./catalog.py.md) — `CatalogConfig`, `A2uiCatalog`

## Exports

- 클래스: `A2uiSchemaManager`

## 상세 명세

### 클래스 `A2uiSchemaManager(InferenceStrategy)`

A2UI 스키마와 카탈로그를 관리하고 LLM 프롬프트를 조립하는 중앙 매니저이다.

#### `__init__(self, version: str, catalogs: Optional[list[CatalogConfig]] = None, accepts_inline_catalogs: bool = False, schema_modifiers: Optional[list[Callable[[dict[str, Any]], dict[str, Any]]]] = None)`

매개변수:
- `version: str` — A2UI 사양 버전 (`"0.8"`, `"0.9"`, `"0.9.1"` 중 하나)
- `catalogs: Optional[list[CatalogConfig]] = None` — 등록할 카탈로그 설정 목록
- `accepts_inline_catalogs: bool = False` — 클라이언트가 보내는 인라인 카탈로그 수락 여부
- `schema_modifiers: Optional[list[Callable[[dict], dict]]] = None` — 스키마 로드 후 적용할 변환 함수 목록

초기화 순서:
1. `_version`, `_accepts_inline_catalogs` 저장.
2. `_server_to_client_schema`, `_common_types_schema`를 `None`으로, `_supported_catalogs`를 빈 리스트로, `_catalog_example_paths`를 빈 dict로, `_schema_modifiers`를 인수 또는 빈 리스트로 초기화.
3. `_load_schemas(version, catalogs or [])` 호출.

#### 프로퍼티 `accepts_inline_catalogs -> bool`

`_accepts_inline_catalogs` 값을 반환한다.

#### 프로퍼티 `supported_catalog_ids -> list[str]`

`_supported_catalogs` 목록의 각 카탈로그에서 `catalog_id`를 추출하여 리스트로 반환한다.

#### 메서드 `_apply_modifiers(self, schema: dict[str, Any]) -> dict[str, Any]`

`_schema_modifiers` 목록을 순서대로 적용하여 스키마를 변환한다. 각 modifier는 `schema -> schema` 형태의 순수 함수여야 한다. modifier가 없으면 원본을 그대로 반환한다.

#### 메서드 `_load_schemas(self, version: str, catalogs: Optional[list[CatalogConfig]] = None)`

버전별 번들 스키마를 로드하고 카탈로그를 처리하는 초기화 단계 메서드이다.

로직:
1. `version`이 `SPEC_VERSION_MAP`에 없으면 `ValueError` 발생.
2. `load_from_bundled_resource(version, SERVER_TO_CLIENT_SCHEMA_KEY, SPEC_VERSION_MAP)`로 s2c 스키마를 로드하고 `_apply_modifiers()`를 적용하여 `_server_to_client_schema`에 저장.
3. 같은 방식으로 common types 스키마를 `_common_types_schema`에 저장.
4. 각 `CatalogConfig`에 대해:
   - `config.provider.load()`로 스키마 로드 후 modifier 적용.
   - `A2uiCatalog` 인스턴스 생성 (`version`, `name`, `catalog_schema`, `s2c_schema`, `common_types_schema`, `custom_cuttable_keys` 포함).
   - `_supported_catalogs`에 추가.
   - `_catalog_example_paths[catalog.catalog_id] = config.examples_path` 저장.

#### 메서드 `_select_catalog(self, client_ui_capabilities: Optional[dict[str, Any]] = None) -> A2uiCatalog`

클라이언트 UI 역량에 따라 적절한 카탈로그를 선택한다.

선택 우선순위:
1. 인라인 카탈로그가 제공되고 허용된 경우: 기본 카탈로그 스키마에 인라인 컴포넌트를 병합한 새 `A2uiCatalog` 반환. 기본 카탈로그는 `supportedCatalogIds`로 먼저 매칭하고, 없으면 `_supported_catalogs[0]`.
2. `supportedCatalogIds`만 제공된 경우: 클라이언트 선호 순서대로 에이전트 지원 카탈로그와 매칭하여 첫 번째 일치 반환.
3. 폴백: `_supported_catalogs[0]` 반환.

에러 케이스:
- `_supported_catalogs`가 비어있으면 `ValueError` 발생.
- `_accepts_inline_catalogs=False`인데 `inlineCatalogs`가 제공되면 `ValueError` 발생.
- `supportedCatalogIds`가 제공되었으나 일치하는 카탈로그가 없으면 `ValueError` 발생.

인라인 카탈로그 병합 상세:
- `copy.deepcopy(base_catalog.catalog_schema)`로 기본 스키마 복사.
- 각 인라인 카탈로그 스키마에 modifier를 적용한 뒤 `CATALOG_COMPONENTS_KEY`(`"components"`) 아래 항목을 `merged_schema["components"].update(...)`로 합친다.
- `name=INLINE_CATALOG_NAME`(`"inline"`)인 새 `A2uiCatalog` 반환.

#### 메서드 `get_selected_catalog(self, client_ui_capabilities: Optional[dict[str, Any]] = None, allowed_components: Optional[list[str]] = None, allowed_messages: Optional[list[str]] = None) -> A2uiCatalog`

`_select_catalog()`로 카탈로그를 선택한 뒤 `catalog.with_pruning(allowed_components, allowed_messages)`를 적용하여 반환한다.

#### 메서드 `load_examples(self, catalog: A2uiCatalog, validate: bool = False) -> str`

`catalog.catalog_id`로 `_catalog_example_paths`에서 예시 경로를 조회한다. 경로가 등록되어 있으면 `catalog.load_examples(path, validate=validate)`를 호출하여 반환한다. 없으면 빈 문자열을 반환한다.

#### 메서드 `generate_system_prompt(self, role_description: str, workflow_description: str = "", ui_description: str = "", client_ui_capabilities: Optional[dict[str, Any]] = None, allowed_components: Optional[list[str]] = None, allowed_messages: Optional[list[str]] = None, include_schema: bool = False, include_examples: bool = False, validate_examples: bool = False) -> str`

LLM에 전달할 최종 시스템 프롬프트를 조립한다.

매개변수:
- `role_description: str` — 에이전트 역할 설명 (프롬프트 첫 부분)
- `workflow_description: str = ""` — 추가 워크플로 지시 (기본 규칙 뒤에 추가)
- `ui_description: str = ""` — UI 설명 섹션
- `client_ui_capabilities: Optional[dict]` — 카탈로그 선택에 사용할 클라이언트 역량
- `allowed_components: Optional[list[str]]` — 포함할 컴포넌트 이름 목록
- `allowed_messages: Optional[list[str]]` — 포함할 메시지 이름 목록
- `include_schema: bool = False` — 카탈로그 스키마 포함 여부
- `include_examples: bool = False` — 예시 포함 여부
- `validate_examples: bool = False` — 예시 로드 시 유효성 검사 여부

조립 순서:
1. `parts = [role_description]`.
2. `workflow = DEFAULT_WORKFLOW_RULES`. `workflow_description`이 있으면 `"\n{workflow_description}"` 추가. `parts.append(f"## Workflow Description:\n{workflow}")`.
3. `ui_description`이 있으면 `parts.append(f"## UI Description:\n{ui_description}")`.
4. `get_selected_catalog(...)`로 카탈로그 선택.
5. `include_schema=True`이면 `selected_catalog.render_as_llm_instructions()`를 `parts`에 추가.
6. `include_examples=True`이면 `load_examples(selected_catalog, validate=validate_examples)`를 호출하고 결과가 있으면 `f"### Examples:\n{examples_str}"`를 추가.
7. `"\n\n".join(parts)` 반환.

## 동작 흐름

1. 생성자에서 번들 스키마를 로드하고 카탈로그 설정을 처리한다.
2. 런타임에 `generate_system_prompt()`가 호출되면 클라이언트 역량에 따른 카탈로그 선택 → 컴포넌트/메시지 가지치기 → 스키마 렌더링 → 예시 로드 → 최종 프롬프트 조립 순서로 진행된다.
3. 인라인 카탈로그 지원 시 클라이언트가 요청마다 다른 컴포넌트 정의를 제공할 수 있어 유연한 UI 렌더링이 가능하다.
