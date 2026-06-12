# agent_sdks/python/a2ui_agent/tests/schema/test_catalog.py

## 개요

`A2uiCatalog`, `CatalogConfig`, `BasicCatalog` 등 카탈로그 관련 핵심 클래스의 동작을 검증하는 pytest 테스트 모음이다. `catalog_id` 프로퍼티 접근, URL 스킴 처리, 버전별 카탈로그 ID 조회 등의 동작이 명세대로 작동하는지 확인한다. 총 6개의 독립적인 테스트 함수로 구성되며 공유 픽스처 없이 직접 인스턴스를 생성해 검증한다.

## 의존성

### 외부 패키지
- `json` (표준 라이브러리)
- `os` (표준 라이브러리)
- `pytest`
- `typing` (`Any`, `Dict`, `List`)

### 저장소 내부 모듈
- [`a2ui.schema.catalog`](../../src/a2ui/schema/catalog.py.md) — `A2uiCatalog`, `resolve_examples_path`, `CatalogConfig`
- [`a2ui.schema.constants`](../../src/a2ui/schema/constants.py.md) — `A2UI_SCHEMA_BLOCK_START`, `A2UI_SCHEMA_BLOCK_END`, `VERSION_0_8`, `VERSION_0_9`
- [`a2ui.basic_catalog.constants`](../../src/a2ui/basic_catalog/constants.py.md) — `BASIC_CATALOG_NAME`
- [`a2ui.basic_catalog`](../../src/a2ui/basic_catalog/__init__.py.md) — `BasicCatalog`

## Exports

이 파일은 테스트 함수만 정의하며, pytest가 수집·실행하는 것 외에 외부에 공개하는 심볼은 없다.

## 테스트 케이스 상세

### `test_catalog_id_property()`

- **검증 동작**: `A2uiCatalog` 인스턴스의 `catalog_id` 프로퍼티가 `catalog_schema` 딕셔너리의 `"catalogId"` 값을 올바르게 반환하는지 확인한다.
- **설정**: `version=VERSION_0_8`, `name=BASIC_CATALOG_NAME`, `s2c_schema={}`, `common_types_schema={}`, `catalog_schema={"catalogId": "https://a2ui.org/specification/v0_9/catalogs/basic/catalog.json"}` 로 `A2uiCatalog`를 생성한다.
- **단언**: `catalog.catalog_id == "https://a2ui.org/specification/v0_9/catalogs/basic/catalog.json"`.
- **픽스처/모킹**: 없음.

### `test_catalog_id_missing_raises_error()`

- **검증 동작**: `catalog_schema`에 `"catalogId"` 키가 없을 때 `catalog_id` 프로퍼티 접근 시 `ValueError`가 발생하고 메시지에 카탈로그 이름이 포함되는지 확인한다.
- **설정**: `catalog_schema={}`(빈 딕셔너리)로 `A2uiCatalog` 생성.
- **단언**: `pytest.raises(ValueError, match=f"Catalog '{BASIC_CATALOG_NAME}' missing catalogId")`.
- **픽스처/모킹**: 없음.

### `test_resolve_examples_path_handling()`

- **검증 동작**: `resolve_examples_path` 함수가 다양한 입력 형식을 올바르게 처리하는지 확인한다.
  - `None` 입력 → `None` 반환.
  - 절대 경로 문자열(`"/absolute/examples"`) → 그대로 반환.
  - `"file:///absolute/examples"` 스킴 → `"/absolute/examples"` 변환 후 반환.
  - `"https://..."` 스킴 → `ValueError("Unsupported examples URL scheme")` 발생.
- **픽스처/모킹**: 없음. 함수를 `from a2ui.schema.catalog import resolve_examples_path`로 테스트 내부에서 지역 임포트.

### `test_catalog_config_from_path_schemes()`

- **검증 동작**: `CatalogConfig.from_path` 클래스 메서드가 다양한 URL 스킴을 올바르게 처리하는지 확인한다.
  - 상대 경로(`"relative_path/to/catalog.json"`) → `config.provider.path == "relative_path/to/catalog.json"`.
  - `"file:///absolute_path/to/catalog.json"` → `config.provider.path == "/absolute_path/to/catalog.json"`.
  - `"http://a2ui.org/catalog.json"` → `NotImplementedError("HTTP support is coming soon.")` 발생.
  - `"ftp://a2ui.org/catalog.json"` → `ValueError("Unsupported catalog URL scheme")` 발생.
- **픽스처/모킹**: 없음. `from a2ui.schema.catalog import CatalogConfig`로 테스트 내부에서 지역 임포트.

### `test_basic_catalog_get_config_examples_path()`

- **검증 동작**: `BasicCatalog.get_config`에 `"file:///"` 스킴의 `examples_path`를 전달했을 때 반환된 `config.examples_path`가 올바르게 스킴 없는 절대 경로로 변환되는지 확인한다.
- **설정**: `version=VERSION_0_9`, `examples_path="file:///absolute/examples"`.
- **단언**: `config.examples_path == "/absolute/examples"`.
- **픽스처/모킹**: 없음. `from a2ui.basic_catalog.provider import BasicCatalog` 및 `from a2ui.schema.constants import VERSION_0_9`로 지역 임포트.

### `test_basic_catalog_id_retrieval_methods()`

- **검증 동작**: `BasicCatalog.get_catalog_id` 정적/클래스 메서드가 버전 문자열에 따라 올바른 카탈로그 ID URL을 반환하는지 확인한다.
  - `"0.8"` → `"https://a2ui.org/specification/v0_8/standard_catalog_definition.json"`.
  - `"0.9"` → `"https://a2ui.org/specification/v0_9/catalogs/basic/catalog.json"`.
  - `"0.7"` → `ValueError("Unsupported version: 0.7")` 발생.
- **픽스처/모킹**: 없음.

## 동작 흐름

각 테스트 함수는 독립적으로 실행되며 공유 픽스처를 사용하지 않는다. 내부적으로 직접 `A2uiCatalog`, `CatalogConfig`, `BasicCatalog` 인스턴스를 생성하거나 함수를 호출한 뒤 반환값과 예외 발생 여부를 `assert` 또는 `pytest.raises` 컨텍스트 매니저로 검증한다. URL 스킴 처리 로직(`file://` 제거, `http`/`ftp` 거부)이 카탈로그 설정 전반에서 일관되게 적용됨을 보장하는 것이 핵심 목적이다.
