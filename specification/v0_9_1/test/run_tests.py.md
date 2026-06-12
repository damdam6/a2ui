# specification/v0_9_1/test/run_tests.py

## 개요

이 파일은 a2ui 명세(v0_9_1)의 JSON Schema 유효성 검사 테스트 러너다. `test/cases/` 디렉토리에 있는 JSON 테스트 스위트 파일들과 JSONL 예시 파일을 읽어 각 데이터를 외부 도구 `ajv`로 검증하고, 전체 통과/실패 수를 집계하여 보고한다. 모든 테스트가 통과하면 정상 종료, 하나라도 실패하면 `sys.exit(1)`로 비정상 종료한다.

## 의존성

### 외부 패키지 / 표준 라이브러리
- `json` — JSON 직렬화·역직렬화
- `subprocess` — `ajv` CLI를 자식 프로세스로 실행
- `os` — 경로 조작 및 파일 존재 여부 확인
- `glob` — 와일드카드 파일 목록 조회
- `sys` — 종료 코드 제어
- `re` (조건부 import, `setup_catalog_alias` 내부) — 정규식으로 `$id` URL 파싱

### 저장소 내부 모듈
없음 (Python import 기준 내부 모듈 의존 없음).

런타임에 참조하는 저장소 내부 파일(경로 상수로 접근):
- `specification/v0_9_1/json/server_to_client.json` — 주 스키마
- `specification/v0_9_1/json/common_types.json` — 공통 타입 스키마
- `specification/v0_9_1/json/client_to_server.json` — 클라이언트→서버 메시지 스키마
- `specification/v0_9_1/catalogs/basic/catalog.json` — 원본 카탈로그 스키마
- `specification/v0_9_1/test/cases/*.json` — 테스트 스위트 파일들
- `specification/v0_9_1/test/cases/contact_form_example.jsonl` — JSONL 예시

## Exports

이 파일은 모듈로 import될 것을 가정하지 않으며 직접 실행용 스크립트다. Python 수준의 공개 export는 없다. 주요 진입점은 `main()` 함수이며 `if __name__ == "__main__"` 가드로 호출된다.

## 상세 명세

### 상수

| 이름 | 값(계산 방식) | 설명 |
|---|---|---|
| `TEST_DIR` | `os.path.dirname(os.path.abspath(__file__))` | 이 스크립트가 위치한 디렉토리의 절대 경로 |
| `SCHEMA_DIR` | `TEST_DIR/../json` 절대화 | JSON 스키마 파일들이 위치한 디렉토리 |
| `CASES_DIR` | `TEST_DIR/cases` | 테스트 케이스 파일들이 위치한 디렉토리 |
| `TEMP_FILE` | `TEST_DIR/temp_data.json` | 개별 테스트 데이터를 임시 저장하는 파일 경로 |
| `TEMP_CATALOG_FILE` | `TEST_DIR/catalog.json` | `setup_catalog_alias`가 생성하는 임시 카탈로그 파일 경로 |

### `SCHEMAS` (dict)

```
{
  "server_to_client.json": <SCHEMA_DIR>/server_to_client.json,
  "common_types.json":     <SCHEMA_DIR>/common_types.json,
  "catalog.json":          TEMP_CATALOG_FILE,  # 임시 생성 파일
  "client_to_server.json": <SCHEMA_DIR>/client_to_server.json,
}
```

스키마 파일명(문자열 키) → 절대 경로 매핑 딕셔너리. 테스트 스위트에서 `"schema"` 필드로 키를 참조하고, `ajv` 호출 시 `-r` 참조 스키마 목록으로도 사용된다.

---

### `setup_catalog_alias() → None`

**목적**: `catalogs/basic/catalog.json`을 읽어 `$id` 값을 수정한 뒤 `TEMP_CATALOG_FILE`에 저장한다. `server_to_client.json`이 `"catalog.json"`이라는 generic URI로 카탈로그를 참조하기 때문에 버전별 `$id`를 범용 형식으로 변환해야 한다.

**동작 단계**:
1. `basic_catalog_path`를 `TEST_DIR/../catalogs/basic/catalog.json` 절대화로 계산한다.
2. 파일이 없으면 오류 메시지 출력 후 `sys.exit(1)`.
3. 파일을 열어 `json.load`로 파싱한다. `JSONDecodeError` 발생 시 오류 출력 후 `sys.exit(1)`.
4. 파싱된 dict에 `"$id"` 키가 있으면, 값을 정규식 `^(https://a2ui\.org/specification/v0_\d+/)` 로 매칭한다.
   - 매칭 성공 시: `catalog["$id"]`를 `<매칭된 그룹1> + "catalog.json"` 으로 교체한다.
   - 매칭 실패 시: `$id`를 그대로 유지한다.
5. 수정된 dict를 `json.dump(..., indent=2)`로 `TEMP_CATALOG_FILE`에 기록한다.

---

### `cleanup_catalog_alias() → None`

**목적**: `TEMP_CATALOG_FILE`이 존재하면 삭제하여 임시 파일을 정리한다.

**동작**: `os.path.exists(TEMP_CATALOG_FILE)` 확인 후 `os.remove` 호출. 파일이 없으면 아무것도 하지 않는다.

---

### `validate_ajv(schema_path: str, data_path: str, all_schemas: dict) → tuple[bool, str]`

**목적**: `ajv` CLI를 subprocess로 실행하여 `data_path`의 JSON 데이터가 `schema_path` 스키마를 통과하는지 검증한다.

**매개변수**:
- `schema_path` — 검증에 사용할 주 스키마의 절대 경로
- `data_path` — 검증할 데이터 JSON 파일의 절대 경로
- `all_schemas` — `SCHEMAS` 딕셔너리. `schema_path`를 제외한 나머지 스키마를 `-r` 참조로 추가하는 데 사용

**동작 단계**:
1. 기본 명령 리스트를 구성한다:
   `["yarn", "run", "ajv", "validate", "-s", schema_path, "--spec=draft2020", "--strict=false", "-c", "ajv-formats", "-d", data_path]`
2. `all_schemas`를 순회하며, 경로가 `schema_path`와 다른 항목마다 `["-r", path]`를 명령에 추가한다.
3. `subprocess.run(cmd, capture_output=True, text=True)`로 실행한다.
4. 반환값: `(returncode == 0, stdout + stderr)` 튜플.
5. `FileNotFoundError` (ajv/yarn 미설치) 발생 시 오류 메시지 출력 후 `sys.exit(1)`.

---

### `run_suite(suite_path: str) → tuple[int, int]`

**목적**: 단일 JSON 테스트 스위트 파일을 실행하고 `(통과 수, 실패 수)` 튜플을 반환한다.

**테스트 스위트 파일 형식** (JSON):
```
{
  "schema": "<SCHEMAS 딕셔너리의 키>",  // 생략 시 기본값 "server_to_client.json"
  "tests": [
    {
      "description": "<설명 문자열>",  // 생략 시 "Test #<인덱스+1>"
      "valid": true | false,            // 생략 시 true
      "data": <임의 JSON 값>
    },
    ...
  ]
}
```

**동작 단계**:
1. `suite_path` 파일을 열어 파싱한다. `JSONDecodeError` 시 오류 출력 후 `(0, 0)` 반환.
2. `suite["schema"]` 키로 `SCHEMAS`에서 스키마 경로를 조회한다. 알 수 없는 키면 오류 출력 후 `(0, 0)` 반환.
3. 스위트명(파일 basename), 테스트 수, 대상 스키마명을 출력한다.
4. `suite["tests"]` 를 순회하며 각 테스트에 대해:
   a. `test["data"]`를 `TEMP_FILE`에 `json.dump`로 기록한다.
   b. `validate_ajv`를 호출하여 `(is_valid, output)`을 얻는다.
   c. `is_valid == expect_valid`이면 `passed` 증가; 아니면 `failed` 증가 후 실패 내용을 출력한다. 실패 시 `is_valid`가 `False`인 경우에만 `output`도 출력한다.
5. `(passed, failed)` 반환.

---

### `validate_jsonl_example(jsonl_path: str) → tuple[int, int]`

**목적**: JSONL 파일의 각 줄을 `server_to_client.json` 스키마로 검증하고 `(통과 수, 실패 수)` 튜플을 반환한다.

**동작 단계**:
1. `jsonl_path` 파일이 존재하지 않으면 오류 출력 후 `(0, 1)` 반환.
2. 파일명(basename)과 대상 스키마명(`server_to_client.json`)을 출력한다.
3. 파일을 줄 단위로 읽으며, 공백 줄은 건너뛴다.
4. 각 비어있지 않은 줄에 대해:
   a. 줄 내용을 `TEMP_FILE`에 텍스트로 그대로 기록한다 (`json.dump` 아닌 `tf.write(line)`).
   b. `validate_ajv`를 호출한다.
   c. 통과/실패를 집계하고, 실패 시 줄 번호와 `output`을 출력한다.
5. `(passed, failed)` 반환.

---

### `main() → None`

**목적**: 전체 테스트 실행 흐름을 조율하는 진입점 함수.

**동작 단계**:
1. `CASES_DIR`이 없으면 메시지 출력 후 반환(종료).
2. `setup_catalog_alias()`를 호출하여 임시 카탈로그 파일을 생성한다.
3. `try` 블록 안에서:
   a. `glob.glob(CASES_DIR/*.json)`으로 테스트 파일 목록을 가져온다.
   b. 파일들을 `sorted()` 순서로 순회하며 `run_suite`를 호출하고 결과를 누적한다.
   c. `cases/contact_form_example.jsonl`에 대해 `validate_jsonl_example`을 호출하고 결과를 누적한다.
   d. `===...===` 구분선과 총 통과/실패 수를 출력한다.
4. `finally` 블록에서:
   - `TEMP_FILE`이 존재하면 삭제한다.
   - `cleanup_catalog_alias()`를 호출한다.
5. `total_failed > 0`이면 `sys.exit(1)`로 종료한다.

## 동작 흐름

```
main()
  └─ setup_catalog_alias()           # catalogs/basic/catalog.json → test/catalog.json ($id 수정)
  └─ glob cases/*.json               # 테스트 스위트 파일 목록 수집
  └─ for each sorted .json file:
       run_suite(file)
         └─ for each test:
              json.dump(data) → TEMP_FILE
              validate_ajv(schema, TEMP_FILE, SCHEMAS)
                └─ subprocess: yarn run ajv validate -s <schema> -d TEMP_FILE -r <others>
  └─ validate_jsonl_example(contact_form_example.jsonl)
       └─ for each line:
            write(line) → TEMP_FILE
            validate_ajv(server_to_client.json, TEMP_FILE, SCHEMAS)
  └─ [finally] rm TEMP_FILE, cleanup_catalog_alias()
  └─ sys.exit(1) if any failures
```

`ajv` CLI는 `yarn run ajv`를 통해 호출되므로, 실행 전 프로젝트 루트에서 `yarn install`이 완료되어야 한다. 모든 검증은 JSON Schema Draft 2020-12(`--spec=draft2020`) 기준이며, `ajv-formats` 플러그인(`-c ajv-formats`)과 비엄격 모드(`--strict=false`)를 사용한다.
