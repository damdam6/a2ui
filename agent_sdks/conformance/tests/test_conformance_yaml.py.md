# agent_sdks/conformance/tests/test_conformance_yaml.py

## 개요

`conformance/suites/` 디렉터리에 있는 모든 YAML 파일이 `conformance_schema.json`에 정의된 JSON Schema를 준수하는지 자동으로 검증하는 pytest 테스트 모듈이다. 각 YAML 파일을 별개의 파라미터화된 테스트 케이스로 실행하여 어느 파일에서 오류가 발생했는지 명확히 식별할 수 있다. 저장소 내부 모듈에 대한 의존성은 없으며 외부 패키지만 사용한다.

## 의존성

### 외부 패키지
- `os` — 파일 경로 조작 및 디렉터리 탐색
- `json` — JSON 파일 파싱
- `yaml` — YAML 파일 파싱 (`PyYAML` 패키지)
- `pytest` — 테스트 프레임워크 및 파라미터화
- `jsonschema` — JSON Schema 기반 데이터 유효성 검증
- `glob` — 와일드카드 패턴으로 파일 목록 수집

### 저장소 내부 모듈
없음

## Exports

이 파일은 테스트 모듈이므로 명시적인 public export는 없다. pytest가 수집하는 항목:
- `test_validate_conformance_yaml` — 파라미터화된 테스트 함수

## 상세 명세

### `load_json_file(path: str) -> Any`
- `path`로 지정된 파일을 UTF-8로 열어 `json.load()`로 파싱한 뒤 Python 객체를 반환한다.
- 에러 처리는 없으며, 파일이 없거나 JSON이 유효하지 않으면 표준 예외가 전파된다.

### `load_yaml_file(path: str) -> Any`
- `path`로 지정된 파일을 UTF-8로 열어 `yaml.safe_load()`로 파싱한 뒤 Python 객체를 반환한다.
- `safe_load`를 사용하므로 임의 Python 객체 역직렬화 위험이 없다.
- 에러 처리는 없으며, 파일이 없거나 YAML이 유효하지 않으면 표준 예외가 전파된다.

### 모듈 수준 상수

| 이름 | 값 / 산출 방식 | 설명 |
|---|---|---|
| `CONFORMANCE_DIR` | `__file__`의 디렉터리(`tests/`)에서 한 단계 위(`..`)로 올라간 절대 경로 | `conformance/` 디렉터리의 절대 경로 |
| `SCHEMA_PATH` | `CONFORMANCE_DIR` + `"conformance_schema.json"` | 스키마 파일의 절대 경로 |
| `SCHEMA` | `load_json_file(SCHEMA_PATH)` 호출 결과 | 모듈 임포트 시 즉시 로드되는 JSON Schema 딕셔너리 |

모듈이 임포트될 때(pytest 수집 단계 포함) `SCHEMA`가 즉시 평가되므로, 스키마 파일이 없거나 파싱 실패 시 수집 단계에서 오류가 발생한다.

### `get_yaml_files() -> list[str]`
- `CONFORMANCE_DIR/suites/*.yaml` 패턴으로 `glob.glob()`을 호출하여 매칭되는 파일 경로 목록을 반환한다.
- `@pytest.mark.parametrize`의 `ids` 인자에 `os.path.basename`이 전달되어, 테스트 ID에는 전체 경로 대신 파일명만 표시된다.

### `test_validate_conformance_yaml(yaml_path: str) -> None`

`@pytest.mark.parametrize("yaml_path", get_yaml_files(), ids=os.path.basename)` 데코레이터로 `get_yaml_files()`가 반환하는 각 경로에 대해 독립적으로 실행된다.

동작 단계:
1. `load_yaml_file(yaml_path)`를 호출해 대상 YAML 파일을 Python 객체로 파싱한다.
2. `os.path.basename(yaml_path)`으로 파일명만 추출해 오류 메시지 식별에 사용한다.
3. `jsonschema.validate(instance=yaml_data, schema=SCHEMA)`를 호출한다.
   - 유효성 검증 성공 시: 함수는 아무런 값 없이 정상 반환되어 테스트가 통과한다.
   - `jsonschema.ValidationError` 발생 시: `pytest.fail()`을 호출하여 `"<basename> failed schema validation: <e.message>"` 형태의 메시지와 함께 테스트를 실패로 표시한다.
4. `ValidationError` 이외의 예외(예: `SchemaError`, 파일 I/O 오류)는 잡지 않으므로 예상치 못한 오류는 그대로 전파된다.

## 동작 흐름

1. **모듈 임포트 / pytest 수집 단계**: `CONFORMANCE_DIR`, `SCHEMA_PATH`가 계산되고, `load_json_file(SCHEMA_PATH)`로 스키마가 메모리에 적재된다. `get_yaml_files()`가 호출되어 `suites/*.yaml` 목록이 생성되고, 각 경로를 파라미터로 하는 테스트 케이스가 등록된다.
2. **테스트 실행 단계**: 등록된 각 `yaml_path`에 대해 `test_validate_conformance_yaml`이 독립적으로 실행된다. YAML 파싱 → JSON Schema 검증 순서로 진행되며, 검증 실패 시 해당 케이스만 실패로 표시되고 다른 케이스는 계속 실행된다.
3. **결과**: `suites/` 아래의 각 YAML 파일 1개가 pytest 테스트 1개에 대응한다. 파일이 추가되거나 삭제되면 pytest 수집 단계에서 자동으로 반영된다.
