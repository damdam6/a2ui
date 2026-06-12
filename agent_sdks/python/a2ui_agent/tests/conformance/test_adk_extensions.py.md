# agent_sdks/python/a2ui_agent/tests/conformance/test_adk_extensions.py

## 개요

`conformance/suites/adk_extensions.yaml` 파일에 정의된 데이터 기반 적합성(conformance) 테스트를 실행하는 파일이다. ADK(Agent Development Kit) 확장과 관련된 `_SendA2uiJsonToClientTool`의 `run_async` 동작을 YAML 케이스에 따라 동기 방식(`asyncio.run`)으로 검증한다. 성공·실패 경로와 에러 메시지 포함 여부를 외부 데이터 파일로 분리하여 관리한다.

## 의존성

### 외부 패키지
- `os`, `yaml`, `json`, `asyncio` (표준 라이브러리)
- `pytest`
- `unittest.mock` — `MagicMock` (테스트 함수 내부 import)

### 저장소 내부 모듈
- [`agent_sdks/python/a2ui_agent/src/a2ui/adk/send_a2ui_to_client_toolset.py`](../../src/a2ui/adk/send_a2ui_to_client_toolset.py.md) — `SendA2uiToClientToolset` (테스트 함수 내부 import)
- [`agent_sdks/python/a2ui_agent/src/a2ui/schema/catalog.py`](../../src/a2ui/schema/catalog.py.md) — `A2uiCatalog` (테스트 함수 내부 import)

## Exports

테스트 함수 및 헬퍼 함수만 포함하며 외부로 내보내는 심볼은 없다.

## 헬퍼 함수 명세

### `_get_conformance_path(filename: str) -> str`
- `__file__` 기준으로 `../../../../conformance/<filename>` 절대 경로를 `os.path.abspath`와 `os.path.join`으로 계산하여 반환한다.

### `load_tests(filename: str) -> list`
- `_get_conformance_path(os.path.join("suites", filename))`으로 경로를 결정하고, UTF-8로 파일을 열어 `yaml.safe_load`로 파싱한 결과 리스트를 반환한다.

### `get_conformance_cases(filename: str) -> list[tuple[str, dict]]`
- `load_tests(filename)`으로 케이스 목록을 로드하고, 각 케이스에서 `(case["name"], case)` 튜플을 만들어 리스트로 반환한다.

## 테스트 케이스 상세 명세

### 모듈 수준 초기화
- `cases_adk_extensions = get_conformance_cases("adk_extensions.yaml")` — 모듈 로드 시 YAML에서 케이스 목록을 미리 수집한다.

### `test_adk_extensions_conformance(name: str, test_case: dict)`
`@pytest.mark.parametrize`로 `cases_adk_extensions`의 각 케이스가 별도 테스트로 실행된다. 현재 구현에서는 `test_case["action"] == "execute_tool"` 단일 분기만 처리한다.

#### action == `"execute_tool"`
1. `args.get("a2ui_json")`이 존재하면 `tool_args = {"a2ui_json": a2ui_json_str}`으로 구성하고, 없으면 `args` 전체를 `tool_args`로 사용한다.
2. `catalog_mock = MagicMock(spec=A2uiCatalog)` 생성, `catalog_mock.validator.validate.return_value = None`으로 설정한다.
3. `_SendA2uiJsonToClientTool(catalog_mock, "examples")` 인스턴스를 생성한다.
4. `tool_context_mock = MagicMock()` 생성, `tool_context_mock.state = {}`, `tool_context_mock.actions = MagicMock(skip_summarization=False)`으로 설정한다.
5. `asyncio.run(tool.run_async(args=tool_args, tool_context=tool_context_mock))`으로 비동기 실행을 동기적으로 수행한다.
6. `expect["success"]`가 `True`인 경우 검증:
   - `"error"` 키가 result에 없어야 한다.
   - `VALIDATED_A2UI_JSON_KEY`가 result에 있어야 한다.
   - `expect.get("contains_validated_json")`이 truthy이면, `json.dumps(result[VALIDATED_A2UI_JSON_KEY])`에 `"beginRendering"` 문자열이 포함되어야 한다.
7. `expect["success"]`가 `False`인 경우 검증:
   - `"error"` 키가 result에 있어야 한다.
   - `expect.get("error_contains")`가 있으면 해당 문자열이 `result["error"]`에 포함되어야 한다.

## 동작 흐름

모듈 로드 시 `adk_extensions.yaml`에서 케이스를 수집하여 parametrize 목록을 구성한다. 각 테스트 실행 시에는 내부 import로 의존성을 로드하고, mock 환경(`catalog_mock`, `tool_context_mock`)을 구성한 뒤 `asyncio.run`을 통해 async 도구 실행을 동기적으로 호출한다. 이후 YAML에 정의된 `expect` 값(success 여부, error_contains, contains_validated_json)과 실제 결과를 비교 검증한다. 모든 케이스는 독립적으로 실행된다.
