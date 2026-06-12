# agent_sdks/python/a2ui_agent/tests/conformance/test_a2a_integration.py

## 개요

`conformance/suites/a2a_integration.yaml` 파일에 정의된 데이터 기반 적합성(conformance) 테스트를 실행하는 파일이다. A2A 통합에 관련된 `create_a2ui_part`, `is_a2ui_part`, `get_a2ui_agent_extension`, `try_activate_a2ui_extension`, `_select_newest_a2ui_extension` 등 공개/비공개 API를 YAML 케이스에 따라 검증한다. 테스트 케이스를 외부 YAML 파일에서 로드하여 `@pytest.mark.parametrize`로 동적으로 실행하는 구조를 갖는다.

## 의존성

### 외부 패키지
- `os`, `re`, `yaml` (표준 라이브러리)
- `pytest`
- `unittest.mock` — `MagicMock` (테스트 함수 내부 import)
- `a2a.types` — `DataPart`, `Part` (테스트 함수 내부 import)
- `a2a.server.agent_execution` — `RequestContext` (테스트 함수 내부 import)

### 저장소 내부 모듈
- [`agent_sdks/python/a2ui_agent/src/a2ui/a2a/parts.py`](../../src/a2ui/a2a/parts.py.md) — `create_a2ui_part`, `is_a2ui_part`, `get_a2ui_datapart` (테스트 함수 내부 import)
- [`agent_sdks/python/a2ui_agent/src/a2ui/a2a/extension.py`](../../src/a2ui/a2a/extension.py.md) — `get_a2ui_agent_extension`, `try_activate_a2ui_extension`, `_select_newest_a2ui_extension` (테스트 함수 내부 import)

## Exports

테스트 함수 및 헬퍼 함수만 포함하며 외부로 내보내는 심볼은 없다.

## 헬퍼 함수 명세

### `_get_conformance_path(filename: str) -> str`
- `__file__` 기준으로 `../../../../conformance/<filename>` 절대 경로를 `os.path.abspath`와 `os.path.join`으로 계산하여 반환한다.

### `load_tests(filename: str) -> list`
- `_get_conformance_path(os.path.join("suites", filename))`으로 경로를 결정하고, UTF-8로 파일을 열어 `yaml.safe_load`로 파싱한 결과(케이스 딕셔너리의 리스트)를 반환한다.

### `get_conformance_cases(filename: str) -> list[tuple[str, dict]]`
- `load_tests(filename)`으로 케이스 목록을 로드하고, 각 케이스에서 `(case["name"], case)` 튜플을 만들어 리스트로 반환한다. parametrize ID 생성에 사용된다.

## 테스트 케이스 상세 명세

### 모듈 수준 초기화
- `cases_a2a_integration = get_conformance_cases("a2a_integration.yaml")` — 모듈 로드 시 YAML에서 케이스 목록을 미리 수집한다.

### `test_a2a_integration_conformance(name: str, test_case: dict)`
`@pytest.mark.parametrize`로 `cases_a2a_integration`의 각 케이스가 별도 테스트로 실행된다. `test_case["action"]` 값에 따라 아래 분기로 처리한다.

#### action == `"create_a2ui_part"`
1. `args["data"]`와 선택적 `args["version"]`으로 `create_a2ui_part(data, version=version)`을 호출한다.
2. `is_a2ui_part(part)`가 truthy인지 확인한다.
3. `get_a2ui_datapart(part).metadata.get("mimeType")`가 `test_case["expect"]["mime_type"]`와 일치하는지 확인한다.

#### action == `"is_a2ui_part"`
1. `args["mime_type"]`으로 `Part(root=DataPart(data={}, metadata={"mimeType": mime_type}))`를 수동 구성한다.
2. `is_a2ui_part(part)` 반환값이 `test_case["expect"]`와 일치하는지 확인한다.

#### action == `"get_extension"`
1. `args["version"]`을 읽고, `args["accepts_inline_catalogs"]`와 `args["supported_catalog_ids"]`가 `None`이 아닌 경우에만 `kwargs`에 추가한다.
2. `get_a2ui_agent_extension(version, **kwargs)`를 호출한다.
3. `ext.uri`가 `expect["uri"]`와 같은지 확인한다.
4. `expect["params"]`가 `None`이면 `ext.params is None`을, 아니면 `ext.params == expect["params"]`를 확인한다.

#### action == `"try_activate"`
1. `args["requested"]`와 `args["advertised"]`를 읽는다.
2. `RequestContext`를 `MagicMock(spec=RequestContext)`으로 생성하고 `context.requested_extensions = requested`, `context.add_activated_extension = MagicMock()`으로 설정한다.
3. `args["advertised"]`의 각 URI 문자열로 mock extension 객체(`ext.uri = uri`)를 만들어 `card.capabilities.extensions` 리스트에 할당한다.
4. `try_activate_a2ui_extension(context, card)` 결과를 받는다.
5. `expect["activated"]`가 `None`이면: `result_version is None`이고 `context.add_activated_extension`이 호출되지 않았음을 검증한다. 아니면: `result_version == expect["version"]`이고 `add_activated_extension`이 `expect["activated"]`로 정확히 1회 호출됨을 검증한다.

#### action == `"select_newest"`
1. `args["requested"]`와 `args["advertised"]`로 `_select_newest_a2ui_extension(requested, advertised)`를 호출한다.
2. 결과가 `expect["newest"]`와 일치하는지 확인한다.

## 동작 흐름

모듈 로드 시 `a2a_integration.yaml`에서 케이스를 수집하여 parametrize 목록을 구성한다. 각 테스트 실행 시에는 `test_case["action"]` 필드를 분기점으로 삼아 해당 A2A 통합 함수를 호출하고, `expect` 필드와 비교 검증한다. 내부 import를 테스트 함수 본문 안에 배치하여 모듈 수준의 불필요한 의존성 로드를 피한다. 픽스처 공유 없이 각 케이스가 독립적으로 실행된다.
