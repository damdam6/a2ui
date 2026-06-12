# specification/scripts/validate.py

## 개요

이 스크립트는 a2ui 저장소의 JSON 스키마 명세(v0_8, v0_9, v1_0)와 해당 예제 파일들을 일괄 검증하는 CLI 도구다. 외부 도구 `ajv`(AJV CLI)를 서브프로세스로 호출하여 JSON Schema Draft 2020 표준에 따른 유효성 검사를 수행하고, 버전별 서브셋 스키마가 표준 스키마의 진부분집합인지도 재귀적으로 비교한다. 모든 버전에서 검증에 실패하면 exit code 1로 종료한다.

## 의존성

### 외부 패키지
- `os` — 경로 조작, 파일 시스템 접근
- `json` — JSON 파싱 및 직렬화
- `subprocess` — ajv CLI 프로세스 실행
- `glob` — 와일드카드 패턴으로 예제 파일 열거
- `sys` — 프로세스 종료 코드 제어
- `shutil` — 임시 디렉터리 삭제

### 저장소 내부 모듈
없음 (순수 스탠드얼론 스크립트)

## Exports

이 파일은 모듈로 import되어 사용되는 것을 전제하지 않는다. 최상위 진입점은 `main()` 함수이며, `__name__ == "__main__"` 가드를 통해 직접 실행 시에만 호출된다.

공개 함수:
- `run_ajv` — 함수
- `validate_messages` — 함수
- `compare_schemas` — 함수
- `main` — 함수

## 상세 명세

### `run_ajv(schema_path, data_paths, refs=None)`

**시그니처:** `(schema_path: str, data_paths: list[str], refs: list[str] | None = None) -> tuple[bool, str]`

**동작 로직:**

1. `__file__`을 기준으로 `../..`를 절대 경로로 계산하여 `repo_root`를 구한다.
2. ajv 실행 파일을 두 위치에서 순서대로 탐색한다:
   - `{repo_root}/node_modules/.bin/ajv`
   - `{repo_root}/specification/v0_9/test/node_modules/.bin/ajv`
   - `next()`를 사용하여 첫 번째로 존재하는 경로를 `local_ajv`에 할당하고, 없으면 `None`.
3. `local_ajv`가 존재하면 해당 경로를 직접 사용하여 명령어를 구성한다:
   `[local_ajv, "validate", "-s", schema_path, "--spec=draft2020", "--strict=false", "-c", "ajv-formats"]`
4. `local_ajv`가 `None`이면 yarn dlx를 폴백으로 사용한다:
   `["yarn", "dlx", "--package=ajv-cli", "--package=ajv-formats", "ajv", "validate", "-s", schema_path, "--spec=draft2020", "--strict=false", "-c", "ajv-formats"]`
5. `refs`가 있으면 각 항목에 대해 `["-r", ref]`를 명령어에 추가한다.
6. `data_paths`의 각 항목에 대해 `["-d", data_path]`를 명령어에 추가한다 (배치 검증).
7. `subprocess.run(cmd, capture_output=True, text=True)`로 실행 후 `returncode == 0` 여부와 `stdout + stderr` 문자열을 튜플로 반환한다.

**반환:** `(성공 여부: bool, 출력 문자열: str)`

---

### `validate_messages(root_schema, example_files, refs=None, temp_dir="temp_val")`

**시그니처:** `(root_schema: str, example_files: list[str], refs: list[str] | None = None, temp_dir: str = "temp_val") -> bool`

**동작 로직:**

1. `os.makedirs(temp_dir, exist_ok=True)`로 임시 디렉터리를 생성한다.
2. `success = True`로 초기화하고, `sorted(example_files)` 순서로 각 파일을 처리한다.
3. 각 예제 파일에 대해:
   - 파일 이름을 출력하며 진행 상황을 표시한다.
   - `json.load()`로 파싱. `json.JSONDecodeError` 발생 시 `[FAIL]` 메시지를 출력하고 `success = False`로 설정 후 `continue`.
   - 파싱된 데이터가 `dict`이고 `"messages"` 키가 있으며 해당 값이 `list`이면 해당 리스트를 `messages`로 사용 (봉투 형식 지원).
   - 그렇지 않고 `list`가 아니면 단일 객체를 `[messages]`로 감싼다.
   - 각 메시지(`msg`)를 `{temp_dir}/msg_{파일명}_{i}.json` 경로의 개별 파일로 직렬화하여 `temp_data_paths` 리스트에 누적한다.
   - `temp_data_paths`가 비어 있으면 `[SKIP]`을 출력하고 건너뛴다.
   - `run_ajv(root_schema, temp_data_paths, refs)`를 호출하여 배치 검증을 수행한다.
   - 검증 실패 시 `[FAIL]`과 ajv 출력을 표시하고 `success = False`.
   - 성공 시 `[PASS]`를 출력한다.
4. 최종 `success` 값을 반환한다.

**반환:** `bool` — 모든 파일 검증 성공 시 `True`, 하나라도 실패 시 `False`

---

### `compare_schemas(subset_path, standard_path)`

**시그니처:** `(subset_path: str, standard_path: str) -> bool`

**동작 로직:**

1. `subset_path`와 `standard_path`를 각각 `json.load()`로 파싱한다. `FileNotFoundError` 또는 `json.JSONDecodeError` 발생 시 `[FAIL]`을 출력하고 `False`를 반환한다.
2. `success = True`로 초기화한다.
3. 승인된 예외 경로 집합 `approved_exceptions`를 정의한다:
   - `"properties.surfaceUpdate.properties.components.items.properties.component.additionalProperties"`
   - `"properties.beginRendering.properties.styles.additionalProperties"`
4. 내부 헬퍼 `get_type_str(val)` — 값 타입을 `"object"`, `"array"`, `"primitive"` 중 하나로 반환한다.
5. 내부 재귀 함수 `compare(sub, std, path="")`:
   - `sub`와 `std`의 타입이 다르면 `[FAIL]`을 출력하고 `success = False` 후 반환.
   - **object인 경우:** `sub`의 모든 키에 대해 `std`에 해당 키가 없으면 실패로 기록하고, 있으면 재귀 호출. (`std`에 여분의 키가 있어도 허용 — 진부분집합 허용)
   - **array인 경우:**
     - `sub`와 `std` 모두 문자열만 포함하면 집합 부분집합 검사 (`set(sub).issubset(set(std))`).
     - 그렇지 않으면 (객체 배열 등) 길이가 다르면 실패, 같으면 인덱스별 재귀 호출.
   - **primitive인 경우:** 값이 다르면 실패 처리. 단, 현재 `path`가 `approved_exceptions`에 포함되어 있으면 무시하고 반환.
6. 최상위에서 `compare(subset, standard)`를 호출하고, `success`가 `True`이면 `[PASS]`를 출력한다.
7. 최종 `success`를 반환한다.

**반환:** `bool`

---

### `main()`

**시그니처:** `() -> None` (실패 시 `sys.exit(1)`)

**동작 로직:**

1. `repo_root`를 `__file__` 기준 `../..`의 절대 경로로 계산한다.
2. `overall_success = True`로 초기화.
3. 버전별 설정 딕셔너리 `configs`를 정의한다 (키: `"v0_8"`, `"v0_9"`, `"v1_0"`). 각 항목의 구조:
   - `root_schema`: 루트 JSON 스키마 경로 (repo_root 기준 상대 경로 문자열)
   - `refs` (선택): 참조 스키마 경로 목록
   - `examples`: 예제 파일 glob 패턴
   - `subset_schema` (v0_8 전용): 서브셋 스키마 경로
4. 각 버전에 대해:
   a. 버전별 임시 디렉터리 `temp_val_{version}`을 생성 (기존 존재 시 `shutil.rmtree`로 먼저 삭제).
   b. `root_schema`가 존재하지 않으면 오류 출력 후 `overall_success = False`로 설정하고 `continue`.
   c. `refs` 처리:
      - `catalog.json`으로 끝나는 ref는 특별 처리: JSON을 읽어 `$id` 필드를 `https://a2ui.org/specification/{version}/catalog.json`으로 덮어쓴 뒤 `version_temp_dir/catalog.json`에 저장하고 해당 경로를 refs에 추가.
      - 나머지 ref는 `repo_root`와 결합한 절대 경로를 그대로 추가.
   d. `config["examples"]` 패턴으로 `glob.glob()`을 실행하여 예제 파일 목록을 얻는다.
   e. `subset_schema`가 설정에 있으면 `compare_schemas()`를 호출하여 스키마 구조 비교를 수행한다.
   f. 예제 파일이 없으면 경고 메시지를 출력하고, 있으면 `validate_messages()`로 검증한다.
   g. 임시 디렉터리를 `shutil.rmtree`로 정리한다.
5. `overall_success`가 `False`이면 `"Overall Validation: FAILED"`를 출력하고 `sys.exit(1)`.
6. 성공이면 `"Overall Validation: PASSED"` 출력 후 정상 종료.

---

### 버전별 `configs` 상수 (내부)

`main()` 내부에서 정의되며 외부로 노출되지 않는다.

| 버전 | root_schema | subset_schema | refs 수 | examples 패턴 |
|---|---|---|---|---|
| `v0_8` | `specification/v0_8/json/server_to_client_with_standard_catalog.json` | `specification/v0_8/json/server_to_client.json` | 0 | `specification/v0_8/json/catalogs/basic/examples/*.json` |
| `v0_9` | `specification/v0_9/json/server_to_client.json` | 없음 | 2 | `specification/v0_9/catalogs/basic/examples/*.json` |
| `v1_0` | `specification/v1_0/json/server_to_client.json` | 없음 | 2 | `specification/v1_0/catalogs/basic/examples/*.json` |

v0_9와 v1_0의 refs: `common_types.json` (절대 경로 그대로), `catalog.json` (별칭 처리 후 임시 경로).

## 동작 흐름

```
main()
  ├─ for version in [v0_8, v0_9, v1_0]:
  │    ├─ 임시 디렉터리 생성
  │    ├─ root_schema 존재 확인
  │    ├─ refs 준비 (catalog.json은 $id 재작성 후 임시 파일로 복사)
  │    ├─ [v0_8만] compare_schemas(subset, root)
  │    │    └─ compare() 재귀 — object/array/primitive 타입별 부분집합 검사
  │    ├─ glob으로 예제 파일 수집
  │    ├─ validate_messages(root_schema, example_files, refs, temp_dir)
  │    │    ├─ 각 파일 JSON 파싱 → 메시지 단위로 분해 → 개별 임시 파일 저장
  │    │    └─ run_ajv(schema, [temp_files], refs) — ajv CLI 배치 호출
  │    └─ 임시 디렉터리 삭제
  └─ overall_success 기반으로 exit(0) 또는 exit(1)
```

핵심 설계 포인트:
- ajv는 배치 모드(`-d` 반복)로 여러 파일을 한 번에 검증하여 프로세스 호출 횟수를 최소화한다.
- catalog.json의 `$id`를 런타임에 표준 URL로 재작성하는 이유는 `server_to_client.json`이 `https://a2ui.org/specification/{version}/catalog.json`이라는 고정 URI로 해당 스키마를 참조하기 때문이다.
- `compare_schemas`의 재귀 비교는 순방향(subset → standard)만 확인하므로 standard에 추가 키/값이 있어도 허용한다.
- `approved_exceptions` 집합은 서브셋이 의도적으로 더 일반적인 값(`additionalProperties` 등)을 사용하는 경로를 화이트리스트로 관리한다.
