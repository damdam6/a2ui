# agent_sdks/kotlin/src/test/kotlin/com/google/a2ui/conformance/ConformanceTestHelper.kt

## 개요

적합성(conformance) 테스트 전반에서 공유되는 유틸리티 싱글턴 객체다. 저장소 루트 디렉터리를 동적으로 탐색하고, 테스트에서 공통으로 사용하는 YAML 키 상수들을 중앙 집중 관리한다. `ConformanceTest.kt`가 이 객체에 의존하며, 파일 경로 계산과 상수 참조를 여기서 가져간다.

## 의존성

### 외부 패키지
- `java.io.File` — 파일시스템 탐색에 사용

### 저장소 내부 모듈
없음

## Exports

- `object ConformanceTestHelper` — 적합성 테스트 공유 유틸리티 싱글턴

## 상세 명세

### `object ConformanceTestHelper`

패키지 `com.google.a2ui.conformance`에 속하는 Kotlin `object` (싱글턴).

#### 필드 / 상수

| 이름 | 종류 | 값 | 설명 |
|---|---|---|---|
| `repoRoot` | `val File` (lazy) | `findRepoRoot()` 결과 | 저장소 루트를 처음 접근 시 한 번만 계산하고 캐시 |
| `SPECIFICATION_DIR` | `private const String` | `"specification"` | 저장소 루트 판별 기준 디렉터리 이름 |
| `CONFORMANCE_DIR_PATH` | `const String` | `"agent_sdks/conformance/"` | 저장소 루트 기준 적합성 테스트 디렉터리 경로 |
| `PROP_USER_DIR` | `private const String` | `"user.dir"` | JVM 시스템 프로퍼티 키 |
| `KEY_NAME` | `const String` | `"name"` | YAML 케이스 이름 키 |
| `KEY_ACTION` | `const String` | `"action"` | YAML 케이스 액션 키 |
| `KEY_ARGS` | `const String` | `"args"` | YAML 케이스 인수 키 |
| `KEY_CATALOG` | `const String` | `"catalog"` | YAML 케이스 카탈로그 키 |
| `KEY_STEPS` | `const String` | `"steps"` | YAML 케이스 스텝 목록 키 |
| `KEY_PAYLOAD` | `const String` | `"payload"` | YAML 스텝 페이로드 키 |
| `KEY_VALIDATE` | `const String` | `"validate"` | YAML 케이스 검증 목록 키 (스텝 대안) |
| `KEY_EXPECT_ERROR` | `const String` | `"expect_error"` | YAML 케이스 예상 오류 키 |
| `KEY_EXPECT` | `const String` | `"expect"` | YAML 케이스 예상 결과 키 |

#### `findRepoRoot(): File` (private)

- `System.getProperty("user.dir")`에서 시작해 부모 디렉터리로 순차적으로 이동한다.
- 각 디렉터리에서 `"specification"` 이름의 하위 디렉터리 존재 여부를 확인한다.
- 발견 즉시 해당 `File`을 반환한다.
- 파일시스템 루트에 도달해도 찾지 못하면 `IllegalStateException("Could not find repository root containing specification directory.")`를 던진다.

#### `getConformanceFile(filename: String): File`

- `File(repoRoot, "$CONFORMANCE_DIR_PATH$filename")`을 반환한다.
- 예: `"suites/validator.yaml"` → `<repo_root>/agent_sdks/conformance/suites/validator.yaml`

#### `getConformanceDir(): File`

- `File(repoRoot, CONFORMANCE_DIR_PATH)`을 반환한다.
- 적합성 테스트의 기준 디렉터리 객체를 제공한다.

## 동작 흐름

테스트 실행 시 처음 `repoRoot`가 접근되는 순간 `findRepoRoot()`가 한 번 실행되어 결과가 캐시된다. 이후 `getConformanceFile` / `getConformanceDir`는 캐시된 루트를 기반으로 경로를 조합해 반환한다. 모든 상수들은 `ConformanceTest.kt` 등 테스트 파일에서 YAML 파싱 시 키 참조용으로 사용된다.
