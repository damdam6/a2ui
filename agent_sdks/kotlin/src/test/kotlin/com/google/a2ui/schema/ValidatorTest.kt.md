# agent_sdks/kotlin/src/test/kotlin/com/google/a2ui/schema/ValidatorTest.kt

## 개요

`A2uiValidator`의 단위 테스트 클래스다. 검증 실패 시 오류 메시지에 정확한 JSON 경로(인덱스 기반 및 컴포넌트 ID 기반)가 포함되는지, 절대/상대 경로를 올바르게 통과시키는지, 잘못된 경로 문법을 정확한 메시지로 거부하는지를 검증한다.

## 의존성

### 외부 패키지
- `kotlin.io.path.createTempFile`
- `kotlin.test.*` (Test, assertEquals, assertFailsWith, assertTrue)
- `kotlinx.serialization.json.*` (Json, JsonArray, JsonObject, JsonPrimitive, jsonObject)

### 저장소 내부 모듈
- `com.google.a2ui.schema.A2uiCatalog` — 카탈로그 데이터 클래스
- `com.google.a2ui.schema.A2uiConstants` — `CATALOG_ID_KEY` 등 상수
- `com.google.a2ui.schema.A2uiValidator` — 검증기 클래스
- `com.google.a2ui.schema.A2uiVersion` — 버전 열거형

## Exports

- `class ValidatorTest` — JUnit 단위 테스트 클래스

## 상세 명세

### 클래스 레벨 픽스처

```
private val simpleCatalog = A2uiCatalog(
  version = A2uiVersion.VERSION_0_8,
  name = "test",
  serverToClientSchema = JsonObject(mapOf("type" to JsonPrimitive("object"))),
  commonTypesSchema = JsonObject(emptyMap()),
  catalogSchema = JsonObject(mapOf(A2uiConstants.CATALOG_ID_KEY to JsonPrimitive("test_id"))),
)
private val pathValidator = A2uiValidator(simpleCatalog)
```

`simpleCatalog`와 `pathValidator`는 인스턴스 변수로, 경로 검증 관련 세 테스트(`validatesAbsolutePathsSuccessfully`, `validatesRelativePathsSuccessfully`, `rejectsInvalidPathsWithUpdatedErrorMessage`)가 공유한다.

## 테스트 케이스 명세

### `reportsPreciseJsonPathsForValidationFailures()`

**검증 동작**: 복잡한 v0.9 스키마로 여러 오류가 있는 페이로드를 검증할 때, 오류 메시지가 정확한 JSON 경로(메시지 인덱스, `updateComponents.components[id='t1']`, `components[1]` 등)를 포함하는지 확인한다.

**픽스처**:
- `s2cSchema`: `CreateSurfaceMessage`, `UpdateComponentsMessage`, `UpdateDataModelMessage`, `DeleteSurfaceMessage`를 `oneOf`로 갖는 인라인 JSON Schema (v0.9).
- `catalogSchema`: `Text`(필수 필드 `component`, `id`, `text`)와 `Image`(필수 필드 `component`, `id`, `url`) 컴포넌트를 정의하는 인라인 카탈로그 스키마.
- `tempCatalogFile`: `catalogSchema`를 임시 파일로 저장 (`createTempFile("catalog", ".json")`), `deleteOnExit()` 등록.
- `A2uiCatalog(version=VERSION_0_9, name="standard", ...)` 생성 후 `schemaMappings = mapOf("catalog.json" to tempCatalogFile.toURI().toString())`.
- 검증 페이로드: 두 메시지 배열
  - 메시지[0]: `createSurface` 에서 `catalogId` 누락
  - 메시지[1]: `updateComponents`의 컴포넌트 배열에 `id="t1"` Text(text 누락, additionalProperty `usageHint`) + 인덱스 1의 Image(url 타입 오류: 123)

**절차**: `validator.validate(payload, strictIntegrity=false)` 호출, `assertFailsWith<IllegalArgumentException>` 기대.

**단언**:
1. `msg.contains("messages[0]: /createSurface")` — 첫 메시지의 createSurface 오류 경로.
2. `msg.contains("messages[1].updateComponents.components[id='t1']")` — ID 기반 컴포넌트 경로.
3. `msg.contains("messages[1].updateComponents.components[1]")` — 인덱스 기반 컴포넌트 경로.

---

### `validatesAbsolutePathsSuccessfully()`

**검증 동작**: `"path": "/absolute/path/to/property"` 형태의 절대 경로를 가진 `updateDataModel` 메시지가 예외 없이 통과하는지 확인한다.

**픽스처**: `pathValidator` (클래스 레벨).

**페이로드**: `version="v0.9"`, `updateDataModel.surfaceId="s1"`, `updateDataModel.value.path="/absolute/path/to/property"`, `value.data="val"`.

**절차**: `pathValidator.validate(payload, strictIntegrity=false)` 호출, 예외가 발생하지 않아야 함.

---

### `validatesRelativePathsSuccessfully()`

**검증 동작**: `"path": "relative/path/to/property"` 형태의 상대 경로를 가진 `updateDataModel` 메시지가 예외 없이 통과하는지 확인한다.

**픽스처**: `pathValidator` (클래스 레벨).

**페이로드**: 절대 경로 테스트와 동일하되 `path` 값이 `"relative/path/to/property"` (앞에 `/` 없음).

**절차**: `pathValidator.validate(payload, strictIntegrity=false)` 호출, 예외 없음 단언.

---

### `rejectsInvalidPathsWithUpdatedErrorMessage()`

**검증 동작**: `"path": "/invalid/escape/~2"` 와 같이 잘못된 이스케이프 시퀀스(`~2`)를 가진 경로가 `IllegalArgumentException`을 발생시키고, 메시지가 정확히 `"Invalid path syntax: '/invalid/escape/~2'"` 인지 확인한다.

**픽스처**: `pathValidator` (클래스 레벨).

**페이로드**: `updateDataModel.value.path="/invalid/escape/~2"`.

**절차**: `assertFailsWith<IllegalArgumentException>` 후 `assertEquals("Invalid path syntax: '/invalid/escape/~2'", exception.message)` 로 메시지 정확성 단언.

## 동작 흐름

`reportsPreciseJsonPathsForValidationFailures`는 임시 파일과 복잡한 스키마를 사용하는 독립적 테스트고, 나머지 세 테스트는 클래스 레벨 `pathValidator`를 공유해 경량 인라인 페이로드를 검증한다. 모든 `validate` 호출에는 `strictIntegrity=false`를 전달한다.
