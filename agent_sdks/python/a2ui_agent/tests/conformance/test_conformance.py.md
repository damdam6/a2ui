# agent_sdks/python/a2ui_agent/tests/conformance/test_conformance.py

## 개요

여러 YAML 기반 적합성(conformance) 테스트 스위트를 단일 파일에서 실행하는 종합 테스트 모듈이다. 스트리밍 파서, 비스트리밍 파서, 검증기(validator), 카탈로그, 스키마 매니저 등 a2ui 코어 컴포넌트의 동작을 `conformance/suites/` 하위의 YAML 파일을 데이터 소스로 삼아 pytest parametrize로 검증한다. 각 스위트는 독립된 `@pytest.mark.parametrize` 함수로 구현된다.

## 의존성

### 외부 패키지
- `os`, `yaml`, `json`, `re` (표준 라이브러리)
- `pytest`

### 저장소 내부 모듈
- [`agent_sdks/python/a2ui_agent/src/a2ui/basic_catalog/__init__.py`](../../src/a2ui/basic_catalog/__init__.py.md) — `BasicCatalog`
- [`agent_sdks/python/a2ui_agent/src/a2ui/schema/catalog.py`](../../src/a2ui/schema/catalog.py.md) — `A2uiCatalog`
- [`agent_sdks/python/a2ui_agent/src/a2ui/parser/streaming.py`](../../src/a2ui/parser/streaming.py.md) — `A2uiStreamParser`
- [`agent_sdks/python/a2ui_agent/src/a2ui/schema/validator.py`](../../src/a2ui/schema/validator.py.md) — `A2uiValidator`
- [`agent_sdks/python/a2ui_agent/src/a2ui/schema/manager.py`](../../src/a2ui/schema/manager.py.md) — `A2uiSchemaManager`, `CatalogConfig`
- [`agent_sdks/python/a2ui_agent/src/a2ui/schema/common_modifiers.py`](../../src/a2ui/schema/common_modifiers.py.md) — `remove_strict_validation`
- [`agent_sdks/python/a2ui_agent/src/a2ui/schema/constants.py`](../../src/a2ui/schema/constants.py.md) — `VERSION_0_8`, `VERSION_0_9`
- [`agent_sdks/python/a2ui_agent/src/a2ui/parser/parser.py`](../../src/a2ui/parser/parser.py.md) — `parse_response`, `has_a2ui_parts` (테스트 함수 내부 import)
- [`agent_sdks/python/a2ui_agent/src/a2ui/parser/payload_fixer.py`](../../src/a2ui/parser/payload_fixer.py.md) — `parse_and_fix` (테스트 함수 내부 import)

## Exports

테스트 함수 및 헬퍼 함수·클래스만 포함하며 외부로 내보내는 심볼은 없다.

## 헬퍼 클래스·함수 명세

### `MemoryCatalogProvider`
- **필드**: `schema` — 생성자 인수를 그대로 저장.
- **`load(self) -> Any`**: `self.schema`를 반환한다. 파일 I/O 없이 메모리의 스키마 딕셔너리를 카탈로그 provider로 제공하기 위한 더미 클래스이다.

### `_get_conformance_path(filename: str) -> str`
- `__file__` 기준으로 `../../../../conformance/<filename>` 절대 경로를 `os.path.abspath`와 `os.path.join`으로 계산하여 반환한다.

### `load_json_file(filename: str) -> Any`
- `_get_conformance_path(filename)`으로 경로를 결정하고, UTF-8로 파일을 열어 `json.load`로 파싱한 결과를 반환한다.

### `load_tests(filename: str) -> list`
- `_get_conformance_path(os.path.join("suites", filename))`으로 경로를 결정하고, UTF-8로 파일을 열어 `yaml.safe_load`로 파싱한 결과 리스트를 반환한다.

### `setup_catalog(catalog_config: dict) -> A2uiCatalog`
- `catalog_config`에서 `"version"`, `"s2c_schema"`, `"catalog_schema"`, `"common_types_schema"`, `"custom_cuttable_keys"`를 읽는다.
- 각 스키마 값이 `str`이면 `load_json_file`로 파일에서 로드하고, `None`이면 빈 딕셔너리 `{}`로 대체한다.
- `custom_cuttable_keys`가 존재하면 `frozenset`으로 변환하고, 없으면 `None`을 전달한다.
- `A2uiCatalog(version=version, name=catalog_config.get("name", "test_catalog"), s2c_schema=..., common_types_schema=..., catalog_schema=..., custom_cuttable_keys=...)`를 생성하여 반환한다.

### `assert_parts_match(actual_parts: list, expected_parts: list)`
- `len(actual_parts) == len(expected_parts)`를 assert한다.
- zip으로 쌍을 순회하며 각 part의 `text`가 `expected.get("text", "")`와 같고, `a2ui_json`이 `expected.get("a2ui")`와 같은지 assert한다.

### `get_conformance_cases(filename: str) -> list[tuple[str, dict]]`
- `load_tests(filename)` 결과 리스트를 `(case["name"], case)` 튜플 목록으로 변환하여 반환한다.

## 테스트 케이스 상세 명세

### `test_parser_conformance(name, test_case)` — `streaming_parser.yaml`
- `setup_catalog(test_case["catalog"])`로 카탈로그를 구성하고 `A2uiStreamParser(catalog=catalog)`를 생성한다.
- `test_case.get("disable_validation")`이 truthy이면 `parser._validator = None`으로 검증을 비활성화한다.
- steps 결정: `test_case["steps"]` → `test_case["process_chunk"]` → `[test_case]` 순으로 폴백한다.
- 각 step에서 `expect_error`(step 또는 test_case에서 조회)가 있으면 `pytest.raises(ValueError, match=expect_error)` 컨텍스트로 `parser.process_chunk(step["input"])`를 호출하고, 없으면 호출 후 `assert_parts_match`로 결과를 검증한다.

### `test_parser_non_streaming_conformance(name, test_case)` — `parser.yaml`
- `action = test_case.get("action", "parse_full")`으로 분기한다. `content = test_case["input"]`.
- **`"parse_full"`**: `parse_response(content)` 호출. `expect_error`가 있으면 `pytest.raises(ValueError)`, 없으면 parts 목록의 길이와 각 part의 `text.strip()` 및 `a2ui_json`을 검증한다.
- **`"fix_payload"`**: `parse_and_fix(content)` 호출. `expect_error`가 있으면 `pytest.raises(ValueError)`, 없으면 `result == test_case["expect"]`를 검증한다.
- **`"has_parts"`**: `has_a2ui_parts(content)` 호출. 반환값이 `test_case["expect"]`와 같은지 검증한다.

### `test_validator_conformance(name, test_case)` — `validator.yaml`
- `setup_catalog(test_case["catalog"])`로 카탈로그를 구성한다.
- steps 결정: `test_case["steps"]` → `test_case["validate"]` → `[test_case]` 순으로 폴백한다.
- 각 step마다 `A2uiValidator(catalog=catalog)`를 새로 생성한다. `expect_error`가 있으면 `pytest.raises(ValueError, match=expect_error)`로 `validator.validate(step["payload"])`를 검증하고, 없으면 예외 없이 완료되는지 확인한다.

### `test_catalog_conformance(name, test_case)` — `catalog.yaml`
- `setup_catalog(test_case["catalog"])`로 카탈로그를 구성한다. `action` 값에 따라 분기한다.
- **`"prune"`**: `allowed_components = args.get("allowed_components", [])`, `allowed_messages = args.get("allowed_messages", [])`를 읽어 `catalog.with_pruning(allowed_components, allowed_messages)` 호출. `expect`에 `"catalog_schema"`, `"s2c_schema"`, `"common_types_schema"` 키가 있으면 각각 비교 검증한다.
- **`"render"`**: `catalog.render_as_llm_instructions().strip()`이 `test_case["expect_output"].strip()`과 같은지 확인한다.
- **`"load"`**: `args["path"]`가 있으면 `os.path.join(__file__ 기준 conformance 디렉토리, path)`로 절대 경로를 구성한다. `expect_error`가 있으면 `pytest.raises`로, 없으면 `catalog.load_examples(full_path, validate=validate)` 결과 `.strip()`이 `expect_output.strip()`과 같은지 확인한다.
- **`"remove_strict_validation"`**: `remove_strict_validation(args["schema"])` 결과가 `test_case["expect"]["schema"]`와 같은지 확인한다.
- **`"verify_cuttable_keys"`**: `set(catalog.cuttable_keys)`가 `set(test_case["expect"]["custom_cuttable_keys"])`와 같은지 확인한다.

### `test_schema_manager_conformance(name, test_case)` — `schema_manager.yaml`
- `action` 값에 따라 분기한다.
- **`"select_catalog"`**: `args["supported_catalogs"]`의 각 항목으로 `CatalogConfig(name=cat_def["catalogId"], provider=MemoryCatalogProvider(cat_def))`를 구성하고, `A2uiSchemaManager(version=VERSION_0_9, catalogs=configs, accepts_inline_catalogs=accepts_inline_catalogs)` 생성 후 `manager.get_selected_catalog(client_capabilities)` 호출. `expect_error`가 있으면 `pytest.raises(ValueError)`, 없으면 `selected.catalog_id`와 `selected.catalog_schema`를 검증한다.
- **`"load_catalog"`**: `catalog_configs`의 각 항목으로 `CatalogConfig.from_path(name=cfg["name"], catalog_path=<절대경로>)` 구성. `modifiers`에 `"remove_strict_validation"`이 있으면 `schema_modifiers` 리스트에 `remove_strict_validation` 함수를 추가. `A2uiSchemaManager(version=VERSION_0_8, catalogs=configs, schema_modifiers=schema_modifiers)` 생성 후 `manager.get_selected_catalog()` 호출하여 `catalog_schema`와 `supported_catalog_ids`(`[c.catalog_id for c in manager._supported_catalogs]`)를 검증한다.
- **`"generate_prompt"`**: `BasicCatalog.get_config(version)`으로 기본 config를 얻고, `examples_path`가 있으면 conformance 디렉토리 기준 절대 경로로 변환한 뒤 `CatalogConfig(name=config.name, provider=config.provider, examples_path=examples_path)`로 감싼다. `A2uiSchemaManager(version=version, catalogs=[config], accepts_inline_catalogs=...)` 생성 후 `manager.generate_system_prompt(role_description, workflow_description, ui_description, include_schema, include_examples, client_ui_capabilities, allowed_components, allowed_messages)` 호출. 결과에서 `re.sub(r"\s+", "", output.strip())`으로 모든 공백을 제거한 뒤, `test_case["expect_contains"]`의 각 문자열(동일하게 공백 제거)이 포함되는지 확인한다.

## 동작 흐름

모듈 로드 시 5개의 YAML 파일(`streaming_parser.yaml`, `parser.yaml`, `validator.yaml`, `catalog.yaml`, `schema_manager.yaml`)에서 케이스를 수집하여 각 parametrize 목록을 구성한다. 각 테스트 실행 시 헬퍼 함수를 통해 카탈로그나 매니저를 구성하고, `action` 필드를 분기점으로 삼아 해당 컴포넌트 API를 호출한 뒤 YAML `expect` 값과 비교한다. `setup_catalog`, `assert_parts_match`, `load_json_file` 등의 공통 헬퍼를 공유하여 코드 중복을 최소화한다.
