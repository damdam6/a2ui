# agent_sdks/python/a2ui_agent/tests/schema/test_schema_manager.py

## 개요

`A2uiSchemaManager` 클래스의 초기화 로직을 검증하는 pytest 테스트 파일이다. 유효한 버전으로의 정상 초기화, 잘못된 버전 입력 시 예외 발생, 그리고 패키지 리소스(`importlib.resources`) 로딩 실패 시 로컬 파일시스템 폴백 동작을 각각 확인한다. `unittest.mock`을 활용한 파일 I/O 모킹이 핵심 기법이다.

## 의존성

### 외부 패키지
- `io` (표준 라이브러리)
- `pytest`
- `json` (표준 라이브러리)
- `os` (표준 라이브러리)
- `unittest.mock` (`patch`, `MagicMock`, `PropertyMock`)

### 저장소 내부 모듈
- [`a2ui.schema.manager`](../../src/a2ui/schema/manager.py.md) — `A2uiSchemaManager`, `A2uiCatalog`, `CatalogConfig`
- [`a2ui.basic_catalog`](../../src/a2ui/basic_catalog/__init__.py.md) — `BasicCatalog`
- [`a2ui.basic_catalog.constants`](../../src/a2ui/basic_catalog/constants.py.md) — `BASIC_CATALOG_NAME`
- [`a2ui.schema.constants`](../../src/a2ui/schema/constants.py.md) — `DEFAULT_WORKFLOW_RULES`, `INLINE_CATALOG_NAME`, `VERSION_0_8`, `VERSION_0_9`, `A2UI_SCHEMA_BLOCK_START`, `A2UI_SCHEMA_BLOCK_END`, `INLINE_CATALOGS_KEY`, `SUPPORTED_CATALOG_IDS_KEY`

## Exports

테스트 함수와 픽스처만 정의하며 외부에 공개하는 심볼은 없다.

## 테스트 케이스 상세

### 픽스처: `mock_importlib_resources`

- **스코프**: 함수 스코프(기본값).
- **동작**: `patch("importlib.resources.files")`를 컨텍스트 매니저로 사용해 `importlib.resources.files` 함수를 `MagicMock`으로 교체하고 해당 mock 객체를 `yield`한다. 테스트 종료 후 패치가 자동 해제된다.
- **용도**: `A2uiSchemaManager` 내부에서 패키지 에셋 로딩에 사용되는 `importlib.resources.files` 호출 경로를 제어하기 위해 사용된다.

### `test_schema_manager_init_valid_version(mock_importlib_resources)`

- **검증 동작**: `VERSION_0_8`과 `BasicCatalog.get_config(VERSION_0_8)`을 인자로 `A2uiSchemaManager`를 생성했을 때 `_server_to_client_schema`와 `_supported_catalogs`가 올바르게 채워지는지 확인한다.
- **모킹 전략**:
  1. `mock_files.side_effect`를 설정해, 인자가 `"a2ui.assets"` 패키지인 경우에만 `mock_traversable`을 반환하고 나머지는 새 `MagicMock`을 반환한다.
  2. `mock_traversable.joinpath.side_effect`를 설정해 경로 인자에 따라 세 가지 결과를 구별 반환한다:
     - 인자가 `VERSION_0_8` 자체이면 `mock_traversable`을 재반환(디렉터리 탐색 체인 지원).
     - `"server_to_client.json"` → `{"$schema": ..., "version": VERSION_0_8, "defs": "server_defs"}` JSON을 `io.StringIO`로 감싼 mock 파일 반환.
     - `"standard_catalog_definition.json"` → `{"$schema": ..., "version": VERSION_0_8, "components": {"Text": {}}}` JSON을 `io.StringIO`로 감싼 mock 파일 반환.
     - 그 외 → `{"$schema": "https://json-schema.org/draft/2020-12/schema"}` JSON만 담긴 mock 파일 반환.
  3. 각 mock 파일은 `.open().__enter__()`가 `io.StringIO` 인스턴스를 반환하도록 구성해 컨텍스트 매니저 프로토콜을 지원한다.
- **단언**:
  - `manager._server_to_client_schema["defs"] == "server_defs"`.
  - `len(manager._supported_catalogs) >= 1`.
  - `manager._supported_catalogs[0].catalog_schema["version"] == VERSION_0_8`.
  - `"Text" in manager._supported_catalogs[0].catalog_schema["components"]`.

### `test_schema_manager_init_invalid_version()`

- **검증 동작**: 알 수 없는 버전 문자열(`"invalid_version"`)을 전달하면 `ValueError("Unknown A2UI specification version")`가 발생하는지 확인한다.
- **픽스처/모킹**: 없음. 실제 `A2uiSchemaManager` 생성자를 직접 호출.

### `test_schema_manager_fallback_local_assets(mock_importlib_resources)`

- **검증 동작**: `importlib.resources.files`가 `FileNotFoundError`를 발생시킬 때 `A2uiSchemaManager`가 로컬 파일시스템 폴백 경로를 통해 정상 초기화되는지 확인한다.
- **모킹 전략**:
  1. `mock_importlib_resources.side_effect = FileNotFoundError("Package not found")`로 패키지 리소스 접근 자체를 실패시킨다.
  2. `patch("os.path.exists", return_value=True)`로 모든 파일 경로 존재 여부를 `True`로 위장한다.
  3. `patch("builtins.open", new_callable=MagicMock)`으로 파일 열기를 가로챈다. `open_side_effect` 내부 분기:
     - 경로 문자열에 `"server_to_client"` 포함 → `{"$schema": ..., "defs": "local_server"}` JSON을 `io.StringIO`로 반환.
     - 경로 문자열에 `"standard_catalog"` 또는 `"catalog"` 포함 → `{"$schema": ..., "catalogId": "basic", "components": {"LocalText": {}}}` JSON을 `io.StringIO`로 반환.
     - 그 외 → `FileNotFoundError` 재발생.
- **단언**:
  - `manager._server_to_client_schema["defs"] == "local_server"`.
  - `len(manager._supported_catalogs) >= 1`.
  - `"LocalText" in manager._supported_catalogs[0].catalog_schema["components"]`.

## 동작 흐름

테스트 파일은 두 가지 핵심 시나리오를 검증한다: (1) 정상 경로 — `importlib.resources`를 통해 패키지 에셋에서 스키마 JSON을 로드하고 `_server_to_client_schema`, `_supported_catalogs`를 구성, (2) 폴백 경로 — 패키지 로딩 실패 시 `os.path.exists` + `open()`을 통해 로컬 파일에서 동일한 데이터를 로드. 두 경우 모두 최종 상태가 동일한 구조를 유지하는지를 단언한다. 버전 검증 테스트는 입력 방어 로직만 별도로 점검한다.
