# agent_sdks/python/a2ui_agent/src/a2ui/basic_catalog/provider.py

## 개요

패키지에 번들된 기본(basic) A2UI 카탈로그를 로드하고 제공하는 모듈이다. `BundledCatalogProvider`가 버전에 맞는 카탈로그 JSON을 패키지 리소스에서 로드하고 필요한 필드를 보완하며, `BasicCatalog`가 이를 래핑해 `CatalogConfig` 및 카탈로그 ID 조회를 위한 정적 인터페이스를 제공한다.

## 의존성

### 외부 패키지
- `typing` — `Any`, `Dict`, `Optional`

### 저장소 내부 모듈
- `..schema.catalog` — `CatalogConfig`, `resolve_examples_path`
- `..schema.catalog_provider` — `A2uiCatalogProvider`
- `..schema.utils` — `load_from_bundled_resource`
- `..schema.constants` — `BASE_SCHEMA_URL`, `CATALOG_ID_KEY`, `CATALOG_SCHEMA_KEY`
- [`.constants`](./constants.py.md) — `BASIC_CATALOG_NAME`, `BASIC_CATALOG_PATHS`

## Exports

- `BundledCatalogProvider` (클래스)
- `BasicCatalog` (클래스)

## 상세 명세

### `BundledCatalogProvider(A2uiCatalogProvider)`

`A2uiCatalogProvider` 추상 클래스를 구현한다. 버전 문자열을 받아 해당 버전의 카탈로그를 패키지 번들 리소스에서 로드한다.

#### `__init__(self, version: str)`

- `self.version`: 사용할 카탈로그 버전 문자열.

#### `load(self) -> Dict[str, Any]`

카탈로그 JSON 딕셔너리를 로드하고 반환한다.

1. `load_from_bundled_resource(self.version, CATALOG_SCHEMA_KEY, BASIC_CATALOG_PATHS)`를 호출해 번들된 JSON을 로드한다.
2. 로드된 딕셔너리에 `CATALOG_ID_KEY`가 없으면, `BASIC_CATALOG_PATHS[version][CATALOG_SCHEMA_KEY]`에서 상대 경로를 가져와 `"/json/"` 부분을 `"/"` 로 치환한 후 `BASE_SCHEMA_URL`에 붙여 ID를 생성하고 삽입한다.
3. `"$schema"` 키가 없으면 `"https://json-schema.org/draft/2020-12/schema"`를 삽입한다.
4. 수정된 딕셔너리를 반환한다.

### `BasicCatalog`

기본 A2UI 카탈로그에 접근하기 위한 정적 헬퍼 클래스. 인스턴스화 없이 정적 메서드로만 사용한다.

#### `@staticmethod get_config(version: str, examples_path: Optional[str] = None) -> CatalogConfig`

기본 번들 카탈로그의 `CatalogConfig`를 반환한다.
- `name`: `BASIC_CATALOG_NAME`(`"basic"`)
- `provider`: `BundledCatalogProvider(version)`
- `examples_path`: `resolve_examples_path(examples_path)` 결과

#### `@staticmethod get_catalog_id(version: str) -> str`

지정된 버전의 기본 카탈로그 ID(URL 형태) 문자열을 반환한다.
- `version`이 `BASIC_CATALOG_PATHS`에 없으면 `ValueError("Unsupported version: {version}")`를 발생시킨다.
- `BASIC_CATALOG_PATHS[version][CATALOG_SCHEMA_KEY]`에서 상대 경로를 가져와 `"/json/"` → `"/"` 치환 후 `BASE_SCHEMA_URL`에 붙여 반환한다.

## 동작 흐름

에이전트가 `BasicCatalog.get_config(version)`을 호출하면 `CatalogConfig`가 생성된다. 이 config를 통해 카탈로그를 초기화하면 `BundledCatalogProvider.load()`가 호출되어 패키지 내 번들된 JSON 파일을 로드하고 필수 메타데이터 필드를 보완한 뒤 반환한다.
