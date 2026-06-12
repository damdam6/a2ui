# agent_sdks/kotlin/src/test/kotlin/com/google/a2ui/conformance/ConformanceTest.kt

## 개요

외부 YAML 파일에 정의된 적합성(conformance) 테스트 스위트를 JUnit 5 `@TestFactory`로 동적 실행하는 클래스다. Validator, Catalog, SchemaManager, Parser, StreamingParser 다섯 영역의 적합성 케이스를 각각의 YAML 파일에서 읽어 실제 구현체를 호출한 결과와 비교한다. 모든 YAML 파싱과 케이스 인스턴스화는 런타임에 수행되며, 테스트 케이스 이름도 YAML에서 가져온다.

## 의존성

### 외부 패키지
- `com.fasterxml.jackson.databind.ObjectMapper` / `com.fasterxml.jackson.dataformat.yaml.YAMLFactory` — YAML 파싱
- `kotlin.test.*` (assertEquals, assertFailsWith, assertNotNull, assertNull, assertTrue)
- `kotlinx.serialization.json.*` (Json, JsonArray, JsonElement, JsonObject, jsonObject)
- `org.junit.jupiter.api.DynamicTest`, `org.junit.jupiter.api.TestFactory`
- `java.io.File`

### 저장소 내부 모듈
- [`ConformanceTestHelper.kt`](ConformanceTestHelper.kt.md) — 저장소 루트 탐색 및 공용 키 상수
- `com.google.a2ui.parser.PayloadFixer` — JSON 페이로드 자동 수정
- `com.google.a2ui.parser.StreamingParser` — 스트리밍 파서 (팩토리)
- `com.google.a2ui.parser.hasA2uiParts` — A2UI 파트 존재 여부 확인 함수
- `com.google.a2ui.parser.parseResponseToParts` — 응답 문자열 파트 파싱 함수
- `com.google.a2ui.schema.A2uiCatalog` — 카탈로그 데이터 클래스
- `com.google.a2ui.schema.A2uiCatalogProvider` — 카탈로그 제공자 인터페이스
- `com.google.a2ui.schema.A2uiSchemaManager` — 스키마 매니저
- `com.google.a2ui.schema.A2uiValidator` — 스키마 검증기
- `com.google.a2ui.schema.A2uiVersion` — 버전 열거형 (VERSION_0_8, VERSION_0_9)
- `com.google.a2ui.schema.CatalogConfig` — 카탈로그 설정 데이터 클래스
- `com.google.a2ui.schema.SchemaModifiers` — 스키마 수정 유틸리티

## Exports

- `class ConformanceTest` — 적합성 테스트 JUnit 클래스 (직접 인스턴스화 불필요)
- `private data class ConformanceTestCase` — 파일 하단에 정의된 테스트 케이스 구조체
- `private data class ValidateStep` — 단일 검증 스텝 구조체

## 상세 명세

### `private data class ConformanceTestCase`
필드: `name: String`, `catalog: A2uiCatalog`, `validate: List<ValidateStep>`, `schemaMappings: Map<String, String>`

### `private data class ValidateStep`
필드: `payload: JsonElement`, `expectError: String?`

---

### `class ConformanceTest`

#### 인스턴스 필드
- `yamlMapper: ObjectMapper` — `ObjectMapper(YAMLFactory())`로 생성, YAML 역직렬화에 사용
- `jsonMapper: ObjectMapper` — `ObjectMapper()`로 생성, Map→JSON 문자열 변환에 사용

#### companion object 상수

| 이름 | 값 |
|---|---|
| `STREAMING_PARSER_YAML_FILE` | `"suites/streaming_parser.yaml"` |
| `SIMPLIFIED_CATALOG_V09` | `"simplified_catalog_v09.json"` |
| `URL_PREFIX_V09` | `"https://a2ui.org/specification/v0_9/"` |
| `URL_PREFIX_V08` | `"https://a2ui.org/specification/v0_8/"` |
| `VERSION_0_8_STR` | `"0.8"` |
| `TEST_CATALOG_NAME` | `"test_catalog"` |
| `VALIDATOR_YAML_FILE` | `"suites/validator.yaml"` |
| `CATALOG_YAML_FILE` | `"suites/catalog.yaml"` |
| `SCHEMA_MANAGER_YAML_FILE` | `"suites/schema_manager.yaml"` |
| `PARSER_YAML_FILE` | `"suites/parser.yaml"` |
| `KEY_EXPECT_CONTAINS` | `"expect_contains"` |
| `KEY_INPUT` | `"input"` |
| `KEY_TEXT` | `"text"` |
| `KEY_A2UI` | `"a2ui"` |
| `KEY_PATH` | `"path"` |
| `KEY_ALLOWED_COMPONENTS` | `"allowed_components"` |
| `KEY_CATALOG_SCHEMA` | `"catalog_schema"` |

---

#### `loadJsonFile(file: File): JsonObject` (private)

`file.readText()`로 파일을 읽고 `Json.parseToJsonElement`로 파싱한 뒤 `JsonObject`로 캐스팅해 반환한다.

---

#### `parseConformanceYaml(file: File, conformanceDir: File): List<ConformanceTestCase>` (private)

YAML 파일을 `yamlMapper.readValue`로 `List<*>`로 역직렬화한 다음, 각 항목을 `ConformanceTestCase`로 변환한다.

1. **스키마 매핑 사전 구성**: `conformanceDir`, `conformanceDir/test_data`, `<repoRoot>/specification/v0_9/json`, `<repoRoot>/specification/v0_8/json` 네 디렉터리에서 `.json` 파일을 탐색한다. 각 파일에 대해 `URL_PREFIX_V09` 및 `URL_PREFIX_V08` 접두사 버전, 그리고 파일명 단독 키 세 가지 항목을 `baseSchemaMappings`에 등록한다.
2. **케이스 파싱**: YAML 항목의 `"name"` 키로 이름을 추출하고, `"catalog"` 키 맵으로 `buildCatalog`를 호출해 `A2uiCatalog`와 추가 `schemaMappings`를 얻는다.
3. **스텝 추출**: `"steps"` → `"validate"` → 최상위에 `"payload"` 키가 있으면 항목 자체를 단일 스텝으로 처리 순으로 폴백한다. 어느 것도 없으면 `IllegalArgumentException`을 던진다.
4. **ValidateStep 생성**: 각 스텝 맵에서 `"payload"` 를 `jsonMapper.writeValueAsString` + `Json.parseToJsonElement`로 `JsonElement`로 변환하고, 스텝 레벨 `"expect_error"` 또는 케이스 레벨 `"expect_error"`를 `expectError`로 설정한다.

---

#### `buildCatalog(catalogMap: Map<*, *>, conformanceDir: File, baseSchemaMappings: Map<String, String>): Pair<A2uiCatalog, Map<String, String>>` (private)

YAML 카탈로그 맵으로부터 `A2uiCatalog` 인스턴스와 확장된 `schemaMappings`를 함께 반환한다.

1. `"version"` 값이 `"0.8"`이면 `A2uiVersion.VERSION_0_8`, 그 외는 `VERSION_0_9`를 선택한다.
2. `"s2c_schema"` 값이 `String`이면 `conformanceDir` 기준 파일을 `loadJsonFile`로 로드하고, `Map`이면 `jsonMapper`로 JSON 문자열 변환 후 `JsonObject`로 파싱하고, 그 외는 빈 `JsonObject`를 사용한다.
3. `"catalog_schema"` 값이 `String`이면 해당 파일을 로드하고 `"catalog.json"` 및 버전 접두사 키를 `schemaMappings`에 추가한다. `Map`이면 임시 파일(`createTempFile("custom_catalog", ".json")`)에 기록하고 `SIMPLIFIED_CATALOG_V09`, `"catalog.json"`, 버전 접두사 키 등 여러 항목을 등록한다. 그 외는 `IllegalArgumentException`을 던진다.
4. `"common_types_schema"` 도 `s2c_schema`와 동일한 방식으로 처리한다.
5. `"custom_cuttable_keys"` 를 `List<String>`으로 추출한다 (없으면 `null`).
6. `A2uiCatalog(version, name=TEST_CATALOG_NAME, serverToClientSchema, commonTypesSchema, catalogSchema, customCuttableKeys)`를 생성해 반환한다.

---

#### `inner class MemoryCatalogProvider(private val schema: JsonObject) : A2uiCatalogProvider`

`load(): JsonObject`를 구현해 생성자에서 받은 `schema`를 그대로 반환한다. 메모리 내 카탈로그 제공자로 테스트 전용이다.

---

#### `@TestFactory testValidatorConformance(): List<DynamicTest>`

`VALIDATOR_YAML_FILE`(`"suites/validator.yaml"`)을 `parseConformanceYaml`로 로드한 뒤, 각 `ConformanceTestCase`에 대해 하나의 `DynamicTest`를 생성한다.

- 각 동적 테스트 내부: `A2uiValidator(case.catalog, case.schemaMappings)`를 생성하고, 케이스의 모든 `ValidateStep`을 순회한다.
- `expectError`가 있으면: `assertFailsWith<IllegalArgumentException>`으로 `validator.validate(payload)` 호출이 실패함을 확인하고, 예외 메시지가 `expectError` 정규식과 일치하거나 `"Validation failed"` 또는 `"Invalid JSON Pointer syntax"`를 포함하는지 검증한다.
- `expectError`가 없으면: `validator.validate(payload)`가 예외 없이 성공해야 한다. 실패 시 `println`으로 로그를 남기고 재던진다.

---

#### `@TestFactory testCatalogConformance(): List<DynamicTest>`

`CATALOG_YAML_FILE`(`"suites/catalog.yaml"`)을 로드해 `action` 값에 따라 분기한다.

- **`"prune"`**: `catalog.withPruning(allowedComponents, allowedMessages)`를 호출하고, `expect` 맵의 `"catalog_schema"`, `"s2c_schema"`, `"common_types_schema"` 항목을 각각 JSON으로 변환해 `assertEquals`로 비교한다.
- **`"load"`**: `catalog.loadExamples(fullPath, validate=validate)`를 호출하고, `expect_error`가 있으면 예외 메시지가 `expectError` 또는 `"Failed to validate example"`을 포함하는지 확인한다. 없으면 출력 문자열을 `expect_output`과 비교한다.
- **`"remove_strict_validation"`**: `args["schema"]`를 `JsonObject`로 변환 후 `SchemaModifiers.removeStrictValidation`을 호출하고, `expect["schema"]`와 결과를 비교한다.
- **`"render"`**: `catalog.renderAsLlmInstructions()`를 호출하고 `expect_output`과 비교한다 (양쪽 trim 적용).
- **`"verify_cuttable_keys"`**: `catalog.cuttableKeys`를 `expect["custom_cuttable_keys"]`와 비교한다.
- 그 외 action: `assert(false)` 호출.

---

#### `@TestFactory testSchemaManagerConformance(): List<DynamicTest>`

`SCHEMA_MANAGER_YAML_FILE`(`"suites/schema_manager.yaml"`)을 로드해 `action` 값에 따라 분기한다.

- **`"select_catalog"`**: `args["supported_catalogs"]` 목록을 `CatalogConfig(name=catalogId, provider=MemoryCatalogProvider(schema))`로 변환하고 `A2uiSchemaManager`를 생성한다. `args["client_capabilities"]`를 `JsonObject`로 변환 후 `manager.getSelectedCatalog(capsJson)`을 호출한다. `expect_error`가 있으면 예외 검증, 없으면 `expect_selected` (카탈로그 ID) 및 `expect_catalog_schema`를 비교한다.
- **`"load_catalog"`**: `case["catalog_configs"]`에서 각 항목의 `"path"`를 `conformanceDir` 기준 절대 경로로 변환해 `CatalogConfig.fromPath`로 생성한다. `modifiers`에 `"remove_strict_validation"`이 있으면 해당 람다를 `schemaModifiers`에 추가한다. `A2uiSchemaManager(VERSION_0_8, ...)`를 생성하고 `expect["catalog_schema"]`, `expect["supported_catalog_ids"]`를 각각 검증한다.
- **`"generate_prompt"`**: `args`에서 `version`, `role_description`, `workflow_description`, `ui_description`, `include_schema`, `include_examples`, `validate_examples`, `client_ui_capabilities`, `allowed_components`, `examples_path`, `accepts_inline_catalogs` 등을 추출한다. `dummyCatalog`(하드코딩된 JSON 리터럴)와 `dummyConfig`를 구성해 `A2uiSchemaManager`를 생성하고 `manager.generateSystemPrompt(...)`를 호출한다. `expect_contains` 목록의 각 항목이 출력에 포함되는지 확인할 때 양쪽 모두 공백을 제거(`Regex("\\s+")` → `""`)해 비교한다.
- 그 외 action: `assert(false)` 호출.

---

#### `@TestFactory testParserConformance(): List<DynamicTest>`

`PARSER_YAML_FILE`(`"suites/parser.yaml"`)을 로드해 `action` 값에 따라 분기한다.

- **`"parse_full"`**: `parseResponseToParts(input)`을 호출한다. `expect_error`가 있으면 예외 메시지가 `expectError` 또는 `"not found in response"`, `"A2UI JSON part is empty"`, `"Failed to parse"` 중 하나를 포함하는지 확인한다. 없으면 결과 파트 수가 `expect` 배열 크기와 같은지, 각 파트의 `text`와 `a2uiJson`이 일치하는지 확인한다. `a2uiJson` 비교에는 `normalizeA2uiJson`을 사용한다.
- **`"fix_payload"`**: `PayloadFixer.parseAndFix(input)`을 호출하고 결과 `JsonArray`를 `expect` JSON과 비교한다.
- **`"has_parts"`**: `hasA2uiParts(input)` 결과를 `expect` Boolean 값과 비교한다.
- 그 외 action: `assert(false)` 호출.

---

#### `@TestFactory testStreamingParserConformance(): List<DynamicTest>`

`STREAMING_PARSER_YAML_FILE`(`"suites/streaming_parser.yaml"`)을 로드한다. 스키마 매핑은 `testValidatorConformance`와 동일한 방식으로 네 디렉터리에서 구성한다. `action`이 `"process_chunk"`가 아닌 케이스는 `return@mapNotNull null`로 건너뛴다.

각 케이스에 대해:
1. `catalogMap`이 있으면 `buildCatalog`로 `(catalog, schemaMappings)`를 구성하고, 없으면 `(null, emptyMap())`을 사용한다.
2. `StreamingParser.create(catalog, schemaMappings)`로 파서를 생성한다.
3. `case["disable_validation"]`이 `true`이면 `parser.validator = null`로 검증을 비활성화한다.
4. `steps` 목록을 순회하며 각 스텝의 `input`으로 `parser.processChunk(input)`을 호출한다.
5. `expect_error`가 있으면 예외 검증 (정규식 또는 `"Failed to parse JSON"`, `"messages["`, 예외 클래스 이름에 `"JsonDecodingException"` 포함 여부 확인).
6. 없으면 결과 파트 수, 각 파트의 `text`, `a2uiJson`을 단계별로 검증한다.

---

#### `normalizeA2uiJson(elem: JsonElement): JsonElement` (private)

재귀적으로 `JsonElement`를 정규화한다. `JsonObject`이면 모든 키-값 쌍에 재귀 적용해 새 `JsonObject`를 반환하고, `JsonArray`이면 모든 요소에 재귀 적용해 새 `JsonArray`를 반환하며, 그 외 원시 타입은 그대로 반환한다. `a2uiJson` 비교 전에 양쪽에 적용해 구조를 정규화한다.

## 동작 흐름

각 `@TestFactory` 메서드는 JUnit 5에 의해 한 번 실행되어 `List<DynamicTest>`를 반환하고, JUnit이 각 `DynamicTest`를 개별 테스트 케이스로 실행한다. YAML 파일 로드 → 케이스 파싱 → 구현체 호출 → 결과 비교의 흐름이다. 내부 헬퍼 `parseConformanceYaml`과 `buildCatalog`는 여러 테스트 팩토리에서 재사용된다.
