# specification/v0_9/test/run_tests.py

## 개요

a2ui v0.9 명세의 JSON Schema 적합성을 자동으로 검증하는 테스트 실행기다. `specification/v0_9/test/cases/` 디렉토리에 있는 JSON 테스트 스위트 파일들을 순서대로 실행하고, 추가로 JSONL 형식의 예제 파일을 한 줄씩 검증한다. 모든 검증은 `ajv` CLI 도구(yarn 스크립트)를 subprocess로 호출하여 수행하며, 한 건이라도 실패하면 exit code 1로 종료한다.

## 의존성

### 외부 패키지 (표준 라이브러리)
- `json` — JSON 파싱 및 직렬화
- `subprocess` — `ajv` CLI 실행
- `os` — 경로 계산 및 파일 존재 확인
- `glob` — 와일드카드 패턴으로 파일 목록 수집
- `sys` — 비정상 종료(`sys.exit`) 및 종료 코드 제어
- `re` (동적 import, `setup_catalog_alias` 내부) — `$id` URL 파싱

### 저장소 내부 모듈
없음 (Python import 기준)

### 런타임 파일 의존성 (경로 참조)
- `../json/server_to_client.json` — 서버→클라이언트 메시지 스키마
- `../json/common_types.json` — 공통 타입 스키마
- `../json/client_to_server.json` — 클라이언트→서버 메시지 스키마
- `../catalogs/basic/catalog.json` — 기본 카탈로그 스키마 원본
- `cases/*.json` — 테스트 스위트 파일들
- `cases/contact_form_example.jsonl` — JSONL 형식 예제 파일

## Exports

이 파일은 스크립트로 직접 실행되며(`if __name__ == "__main__"`) 외부에 공개하는 Python 심볼은 없다.

## 상세 명세

### 상수

| 이름 | 값 (계산 방식) | 설명 |
|------|--------------|------|
| `TEST_DIR` | `os.path.dirname(os.path.abspath(__file__))` | 이 스크립트가 위치한 디렉토리의 절대 경로 |
| `SCHEMA_DIR` | `TEST_DIR/../json` (절대화) | JSON Schema 파일들이 위치한 디렉토리 |
| `CASES_DIR` | `TEST_DIR/cases` | 테스트 케이스 파일들이 위치한 디렉토리 |
| `TEMP_FILE` | `TEST_DIR/temp_data.json` | ajv 검증 시 데이터를 임시 저장하는 파일 |
| `TEMP_CATALOG_FILE` | `TEST_DIR/catalog.json` | `$id`를 수정한 임시 카탈로그 스키마 파일 |

### `SCHEMAS` 딕셔너리

```
{
  "server_to_client.json": <SCHEMA_DIR>/server_to_client.json,
  "common_types.json":     <SCHEMA_DIR>/common_types.json,
  "catalog.json":          TEMP_CATALOG_FILE,
  "client_to_server.json": <SCHEMA_DIR>/client_to_server.json,
}
```

스키마 파일명(문자열 키)에서 실제 절대 경로로의 매핑이다. `catalog.json` 키는 원본 파일이 아닌 임시 파일(`TEMP_CATALOG_FILE`)을 가리키며, 테스트 실행 전에 `setup_catalog_alias()`가 이 파일을 생성한다.

---

### `setup_catalog_alias() -> None`

**역할**: `../catalogs/basic/catalog.json`을 읽어 `$id` 필드를 정규화한 뒤 `TEMP_CATALOG_FILE`로 저장하는 전처리 함수다.

**동작 로직**:
1. `basic_catalog_path`를 `TEST_DIR/../catalogs/basic/catalog.json`의 절대 경로로 계산한다.
2. 파일이 존재하지 않으면 오류 메시지 출력 후 `sys.exit(1)`.
3. 파일을 열어 `json.load()`로 파싱한다. `JSONDecodeError` 발생 시 오류 메시지 출력 후 `sys.exit(1)`.
4. 파싱된 딕셔너리에 `"$id"` 키가 존재할 경우, 정규식 `r"^(https://a2ui\.org/specification/v0_\d+/)"` 로 접두사를 추출한다. 일치하면 `$id`를 `<prefix>catalog.json` 형태로 교체한다. 이는 `server_to_client.json`이 참조하는 `"catalog.json"` URI와 일치시키기 위함이다.
5. 수정된 딕셔너리를 `TEMP_CATALOG_FILE`에 `indent=2`로 직렬화하여 저장한다.

---

### `cleanup_catalog_alias() -> None`

**역할**: `setup_catalog_alias()`가 생성한 임시 파일(`TEMP_CATALOG_FILE`)을 삭제하는 정리 함수다.

**동작 로직**:
1. `TEMP_CATALOG_FILE`이 존재하면 `os.remove()`로 삭제한다. 없으면 아무것도 하지 않는다.

---

### `validate_ajv(schema_path: str, data_path: str, all_schemas: dict) -> tuple[bool, str]`

**역할**: 주어진 스키마와 데이터 파일 경로로 `ajv validate` 명령을 subprocess로 실행하고 통과 여부와 출력 문자열을 반환한다.

**매개변수**:
- `schema_path` — 검증에 사용할 기본 스키마의 절대 경로
- `data_path` — 검증할 데이터 JSON 파일의 절대 경로
- `all_schemas` — `SCHEMAS` 딕셔너리 전체 (참조 스키마로 추가되는 나머지 스키마들 포함)

**반환**: `(is_valid: bool, output: str)` 튜플. `is_valid`는 `returncode == 0`이면 `True`.

**동작 로직**:
1. 기본 명령을 구성한다: `["yarn", "run", "ajv", "validate", "-s", schema_path, "--spec=draft2020", "--strict=false", "-c", "ajv-formats", "-d", data_path]`
2. `all_schemas`를 순회하며 `path != schema_path`인 모든 항목에 대해 `-r <path>` 인수를 명령에 추가한다 (참조 스키마 등록).
3. `subprocess.run()`을 `capture_output=True, text=True` 옵션으로 호출한다.
4. `FileNotFoundError`가 발생하면 (yarn/ajv 미설치) 오류 메시지 출력 후 `sys.exit(1)`.
5. `(returncode == 0, stdout + stderr)` 를 반환한다.

---

### `run_suite(suite_path: str) -> tuple[int, int]`

**역할**: 단일 테스트 스위트 JSON 파일을 읽어 포함된 모든 테스트 케이스를 실행하고 통과/실패 수를 반환한다.

**매개변수**:
- `suite_path` — 테스트 스위트 파일의 절대 경로

**반환**: `(passed: int, failed: int)` 튜플.

**테스트 스위트 파일 구조** (JSON):
```
{
  "schema": "<SCHEMAS 딕셔너리의 키>",   // 선택, 기본값 "server_to_client.json"
  "tests": [
    {
      "description": "<설명>",           // 선택, 기본값 "Test #<번호>"
      "valid": true | false,             // 선택, 기본값 true (유효해야 하는지)
      "data": { ... }                    // 검증할 JSON 데이터
    },
    ...
  ]
}
```

**동작 로직**:
1. `suite_path`를 열어 `json.load()`로 파싱한다. `JSONDecodeError` 발생 시 `(0, 0)` 반환.
2. `suite["schema"]` 값(기본값 `"server_to_client.json"`)이 `SCHEMAS`에 없으면 오류 출력 후 `(0, 0)` 반환.
3. 스위트 이름(파일 basename), 테스트 수, 대상 스키마를 출력한다.
4. `tests` 리스트를 순회하며 각 케이스에 대해:
   a. `data` 값을 `TEMP_FILE`에 `json.dump()`로 기록한다.
   b. `validate_ajv(schema_path, TEMP_FILE, SCHEMAS)`를 호출한다.
   c. `is_valid == expect_valid`이면 `passed` 증가; 그렇지 않으면 `failed` 증가하고 설명, 기대값/실제값, (검증 실패 시) ajv 출력을 출력한다.
5. `(passed, failed)` 반환.

---

### `validate_jsonl_example(jsonl_path: str) -> tuple[int, int]`

**역할**: JSONL 파일의 각 줄을 `server_to_client.json` 스키마로 검증하고 통과/실패 수를 반환한다.

**매개변수**:
- `jsonl_path` — 검증할 JSONL 파일의 절대 경로

**반환**: `(passed: int, failed: int)` 튜플.

**동작 로직**:
1. 파일이 존재하지 않으면 오류 메시지 출력 후 `(0, 1)` 반환.
2. 파일 basename과 대상 스키마(`server_to_client.json`)를 출력한다.
3. 파일을 열어 줄 단위로 순회한다.
   - 빈 줄(`strip()` 후 빈 문자열)은 건너뛴다.
   - 각 줄을 `TEMP_FILE`에 그대로 기록한다(JSON 재파싱 없이 원문 사용).
   - `validate_ajv(schema_path, TEMP_FILE, SCHEMAS)` 호출.
   - 통과 시 `passed` 증가, 실패 시 `failed` 증가하고 줄 번호와 ajv 출력을 출력한다.
4. `(passed, failed)` 반환.

---

### `main() -> None`

**역할**: 전체 테스트 실행의 진입점으로, 전처리 → 스위트 실행 → JSONL 검증 → 정리 → 결과 출력 순으로 진행한다.

**동작 로직**:
1. `CASES_DIR`이 존재하지 않으면 메시지 출력 후 즉시 반환(종료 코드 0).
2. `setup_catalog_alias()`를 호출하여 임시 카탈로그 파일을 생성한다.
3. `try` 블록 내에서:
   a. `glob.glob(CASES_DIR/*.json)`으로 테스트 파일 목록을 수집한다.
   b. 정렬된 파일 목록에 대해 `run_suite()`를 호출하여 `total_passed`, `total_failed`를 누적한다.
   c. `cases/contact_form_example.jsonl`에 대해 `validate_jsonl_example()`을 호출하여 누적한다.
   d. 구분선(`="*30`)과 총 통과/실패 수를 출력한다.
4. `finally` 블록에서 `TEMP_FILE`과 임시 카탈로그 파일(`cleanup_catalog_alias()`)을 삭제한다.
5. `total_failed > 0`이면 `sys.exit(1)`로 비정상 종료한다.

## 동작 흐름

```
main()
  ├─ setup_catalog_alias()         # 임시 catalog.json 생성
  ├─ [try]
  │   ├─ glob cases/*.json
  │   ├─ for each .json → run_suite()
  │   │     ├─ 스키마 결정
  │   │     └─ for each test
  │   │           ├─ data → TEMP_FILE
  │   │           └─ validate_ajv() → subprocess("yarn run ajv validate ...")
  │   ├─ validate_jsonl_example(contact_form_example.jsonl)
  │   │     └─ for each line
  │   │           ├─ line → TEMP_FILE
  │   │           └─ validate_ajv()
  │   └─ 결과 출력
  ├─ [finally] TEMP_FILE 삭제, cleanup_catalog_alias()
  └─ total_failed > 0 → sys.exit(1)
```

전체적으로 "임시 파일에 데이터를 쓰고 ajv subprocess를 호출한 뒤 반환 코드로 합격 여부를 판정"하는 패턴이 반복된다. 임시 파일은 `finally`로 반드시 정리되며, 카탈로그 별칭 파일도 동일하게 보장된다. 최종 종료 코드는 CI 파이프라인에서 실패를 감지할 수 있도록 실패 건수가 1 이상이면 1, 그렇지 않으면 0이다.
