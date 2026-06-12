# agent_sdks/kotlin/src/main/kotlin/com/google/a2ui/schema/Validator.kt

## 개요

A2UI JSON 페이로드의 구조적 유효성과 위상적 무결성을 검증하는 핵심 클래스 파일이다. `A2uiValidator` 공개 클래스와 두 개의 비공개 내부 클래스(`A2uiTopologyValidator`, `A2uiRecursionValidator`)로 구성된다. `networknt/json-schema-validator` 라이브러리를 사용하여 JSON Schema Draft 2020-12 기반 스키마 유효성 검사를 수행하고, 추가로 컴포넌트 그래프 위상 검사(고아·사이클·누락 참조)와 재귀 깊이 제한 검사를 적용한다. VERSION_0_8과 VERSION_0_9(이상)에 대해 각기 다른 검증 전략을 사용한다.

## 의존성

### 외부 패키지
- `com.networknt.schema.InputFormat` — JSON 문자열 입력 형식 지정
- `com.networknt.schema.Schema` — 스키마 검증기 타입
- `com.networknt.schema.SchemaRegistry` — 스키마 레지스트리 팩토리
- `com.networknt.schema.SchemaRegistryConfig` — 레지스트리 설정 빌더
- `com.networknt.schema.dialect.Dialects` — JSON Schema 방언(Draft 지정)
- `kotlinx.serialization.json.*` — JSON 요소 타입 및 직렬화

### 저장소 내부 모듈
- [`A2uiCatalog`, `A2uiVersion`](Catalog.kt.md) — 카탈로그 데이터 클래스 및 버전 열거형
- [`A2uiConstants`](A2uiConstants.kt.md) — `CATALOG_COMPONENTS_KEY`, `CATALOG_STYLES_KEY`, `BASE_SCHEMA_URL` 등 상수
- [`SchemaResourceLoader`](SchemaResourceLoader.kt.md) — `wrapAsJsonArray` 유틸리티
- [`TopologyAnalyzer`](TopologyAnalyzer.kt.md) — 컴포넌트 참조 필드 추출 및 위상 분석
- `SchemaInspector` — [`SchemaInspector.kt`](SchemaInspector.kt.md) — `getComponentReferences`, `updateAdjacencyList`, `visit`
- 패키지 내부 상수 (`PROP_PROPERTIES`, `PROP_ITEMS`, `PROP_COMPONENT`, `PROP_ADDITIONAL_PROPERTIES` 등) — 각각 `SchemaInspector.kt` 및 `Validator.kt` 내 companion object에 정의

## Exports

| 이름 | 종류 |
|------|------|
| `A2uiValidator` | class (public) |
| `A2uiValidator.validate` | 함수 (public) |

## 상세 명세

### class A2uiValidator

`@JvmOverloads` 어노테이션이 붙은 주 생성자를 가지며, Java 코드에서 `schemaMappings` 기본값 생략 호출이 가능하다.

#### 생성자 매개변수
- `catalog: A2uiCatalog` — 유효성 검사에 사용할 카탈로그 (스키마 포함).
- `schemaMappings: Map<String, String> = emptyMap()` — URI 프리픽스 매핑 (스키마 ID 리다이렉션에 사용).

#### 필드 (private)
- `shared0_9Registry: SchemaRegistry` (by lazy) — VERSION_0_9용 공유 레지스트리. `Dialects.getDraft202012()`로 초기화하며 `schemaMappings`의 각 `(prefix, target)` 쌍을 `schemaIdResolvers.mapPrefix`로 등록한다.
- `sharedConfig: SchemaRegistryConfig` (by lazy) — `SchemaRegistryConfig.builder().build()`로 생성되는 기본 설정.
- `subValidators: MutableMap<String, Schema>` — 서브 스키마 검증기 캐시 (컴포넌트 타입명 또는 정의명을 키로 사용).
- `validator: Schema` — `buildValidator()` 호출 결과로 초기화되는 최상위 스키마 검증기.

#### private fun buildValidator(): Schema

`catalog.version == A2uiVersion.VERSION_0_8`이면 `build0_8Validator()`를, 그 외이면 `build0_9Validator()`를 호출하여 반환한다.

#### private fun injectAdditionalProperties(schema: JsonElement, sourceProperties: Map<String, JsonElement>): Pair<JsonElement, Set<String>>

스키마 트리를 재귀적으로 순회하여 `additionalProperties: true`로 설정된 `JsonObject` 노드에 `sourceProperties`의 해당 키 값을 주입한다.

동작 단계:
1. 내부 재귀 함수 `recursiveInject(obj: JsonElement): JsonElement`를 정의한다.
2. `obj`가 `JsonObject`이면 각 `(k, v)` 쌍을 순회. `v`가 `JsonObject`이고 `v["additionalProperties"]`가 boolean `true`일 때:
   - `sourceProperties`에 키 `k`가 있으면: `injectedKeys.add(k)`, 해당 노드의 `additionalProperties`를 `false`로 교체, `properties`를 기존 값과 `sourceProperties[k]`의 합집합으로 교체.
   - `sourceProperties`에 키 `k`가 없으면: 해당 `v`에 재귀 호출.
3. `obj`가 `JsonArray`이면 각 원소에 재귀 호출하여 새 배열 반환.
4. 그 외이면 `obj` 그대로 반환.
5. 최종적으로 `recursiveInject(schema) to injectedKeys`를 반환한다.

#### private fun bundle0_8Schemas(): JsonObject

VERSION_0_8용 통합 스키마를 구성한다.

동작 단계:
1. `catalog.serverToClientSchema.isEmpty()`이면 빈 `JsonObject`를 반환한다.
2. `catalog.catalogSchema`에서 `components` 키 값을 `sourceProperties["component"]`로, `styles` 키 값을 `sourceProperties["styles"]`로 저장한다 (null 안전 처리).
3. `injectAdditionalProperties(catalog.serverToClientSchema, sourceProperties)`를 호출하여 통합 스키마를 얻고 `JsonObject`로 캐스팅하여 반환한다.

#### private fun build0_8Validator(): Schema

1. `bundle0_8Schemas()`로 통합 스키마 생성.
2. `SchemaResourceLoader.wrapAsJsonArray(bundledSchema)`로 배열 스키마로 래핑 후 `mutableMap`으로 변환.
3. `"$schema"` 키를 `"https://json-schema.org/draft/2020-12/schema"`로 설정.
4. `catalog.serverToClientSchema["$id"]`에서 `baseUri`를 추출 (없으면 `A2uiConstants.BASE_SCHEMA_URL`). `baseUri`의 마지막 `/` 이전까지를 `baseDirUri`로, `"$baseDirUri/common_types.json"`을 `commonTypesUri`로 계산.
5. `Dialects.getDraft202012()` 기반 레지스트리를 구성하되 `schemaMappings` 프리픽스와 `"common_types.json"` → `commonTypesUri` 매핑을 등록한다.
6. 완성된 스키마를 JSON 문자열로 직렬화하여 `registry.getSchema(schemaString, InputFormat.JSON)`로 `Schema`를 반환한다.

#### private fun build0_9Validator(): Schema

1. `SchemaResourceLoader.wrapAsJsonArray(catalog.serverToClientSchema)`로 배열 스키마 생성 후 mutableMap 변환.
2. `"$schema"` 키 설정 후 JSON 직렬화.
3. `shared0_9Registry.getSchema(schemaString, InputFormat.JSON)`로 `Schema`를 반환한다.

#### fun validate(a2uiJson: JsonElement, strictIntegrity: Boolean = true)

A2UI JSON 페이로드에 대해 스키마 유효성 + 무결성 검사를 수행한다. 오류 시 `IllegalArgumentException`을 던진다.

동작 단계:
1. `a2uiJson`을 `JsonArray`로 캐스팅 시도. 실패하면 단일 원소 `JsonArray`로 감싼다.
2. VERSION_0_9이면 `validate0_9Custom(messages, strictIntegrity)`를 호출하고 이 단계를 건너뜀. 그 외이면 JSON 문자열로 직렬화 후 `validator.validate(messagesString, InputFormat.JSON)`로 오류를 수집하고, 오류가 있으면 메시지를 조합한 `IllegalArgumentException`을 던진다.
3. `strictIntegrity`가 `true`이면:
   a. `calculateSurfaceRootIds(messages)`로 `surfaceId → rootId` 맵을 구성한다.
   b. 각 메시지에서 `surfaceUpdate` 또는 `updateComponents` 키를 찾아 `surfaceId`와 `components` 배열을 추출한다.
   c. 해당 `surfaceId`의 `rootId`를 맵에서 조회하여 `A2uiTopologyValidator(catalog, rootId).validate(components)`를 호출한다.
4. 모든 메시지에 대해 `A2uiRecursionValidator(strictIntegrity).validate(message)`를 호출한다.

#### private fun validate0_9Custom(messages: JsonArray, strictIntegrity: Boolean)

VERSION_0_9 메시지 배열을 메시지 타입별로 분기 검증한다.

동작 단계:
1. `allErrors: MutableList<String>` 초기화.
2. 각 메시지(`messageElem`)에 대해 `JsonObject`가 아니면 `"$basePath: Is not an object"` 오류 추가 후 `continue`.
3. 메시지에 포함된 최상위 키에 따라 분기:
   - `"createSurface"` → `getSubValidator("CreateSurfaceMessage")`로 검증
   - `"updateComponents"` → `getUpdateComponentsErrors(messageElem, basePath)` 호출
   - `"updateDataModel"` → `getSubValidator("UpdateDataModelMessage")`로 검증
   - `"deleteSurface"` → `getSubValidator("DeleteSurfaceMessage")`로 검증
   - 그 외 → `"$basePath: Unknown message type with keys $keys"` 오류 추가
4. `allErrors`가 비어 있지 않으면 전체 오류를 조합한 `IllegalArgumentException`을 던진다.

#### private fun getSubValidator(defName: String): Schema

`subValidators` 캐시에서 `defName`에 대한 `Schema`를 반환하거나 새로 생성하여 캐시에 저장한다.

동작 단계:
1. `catalog.serverToClientSchema["$defs"]`를 `JsonObject`로 가져온다. 없으면 `IllegalArgumentException("No \$defs found in schema")`.
2. `defs[defName]`을 가져온다. 없으면 `IllegalArgumentException("Definition $defName not found in schema")`.
3. `{"$schema": SCHEMA_DRAFT_2020_12, "$defs": defs, "$ref": "#/$defs/$defName"}` 구조의 임시 스키마를 구성한다.
4. JSON 직렬화 후 `shared0_9Registry.getSchema(...)`로 `Schema`를 반환한다.

#### private fun getFormattedErrors(validator: Schema, instance: JsonElement, basePath: String): List<String>

인스턴스를 JSON 직렬화하여 `validator.validate`를 실행하고, 오류 메시지를 사람이 읽기 좋은 형식으로 변환하여 반환한다.

동작 단계:
1. 각 오류 메시지 문자열에 대해 정규식 `"property '(.*?)' is not defined in the schema and the schema does not allow additional properties"`로 매칭을 시도한다. 매칭되면 `"$basePath: '$prop' was unexpected"` 형식으로 변환.
2. 매칭 실패 시 `: `, `$.`, `$` 접두어를 제거하고 `"$basePath: $cleanMsg"` 형식으로 변환.

#### private fun getUpdateComponentsErrors(message: JsonObject, path: String): List<String>

`updateComponents` 메시지의 상세 유효성을 직접 검사하여 오류 목록을 반환한다.

검사 항목:
1. `message["version"]`이 `"v0.9"`인지 확인. 아니면 `"$path: Invalid version, expected 'v0.9'"` 추가.
2. `message["updateComponents"]`가 `JsonObject`인지 확인. 아니면 오류 추가 후 즉시 반환.
3. `ucElem["surfaceId"]`가 문자열 `JsonPrimitive`인지 확인.
4. `ucElem["components"]`가 `JsonArray`인지 확인. 아니면 오류 추가 후 즉시 반환.
5. 컴포넌트 ID 중복 검사: `groupingBy { it }.eachCount().filter { it.value > 1 }.keys`로 중복 ID 집합 확인.
6. 각 컴포넌트 원소에 대해 `getSingleComponentErrors(compElem, compPath)` 호출하여 오류 누적.

#### private fun getSingleComponentErrors(comp: JsonObject, path: String): List<String>

단일 컴포넌트의 타입을 판별하고 해당 카탈로그 스키마로 검증한다.

동작 단계:
1. `comp["component"]?.jsonPrimitive?.content`로 `compType` 추출. 없으면 `"$path: Missing 'component' field"` 반환.
2. `catalog.catalogSchema["components"]`를 가져온다. 없으면 오류 반환.
3. `componentsObj[compType]`로 해당 컴포넌트 스키마 조회. 없으면 `"$path: Unknown component: $compType"` 반환.
4. `subValidators` 캐시에서 `"comp_$compType"` 키로 서브 검증기를 조회하거나, `catalogSchema` 전체를 기반으로 `{"$schema": ..., "$ref": "#/components/$compType"}` 임시 스키마를 구성하여 `shared0_9Registry`에서 생성한다.
5. `getFormattedErrors(validator, comp, path)`를 호출하여 반환한다.

#### private fun calculateSurfaceRootIds(messages: JsonArray): Map<String, String>

메시지 배열을 순회하여 `surfaceId → rootId` 맵을 구성한다.

동작 단계:
1. `"beginRendering"` 메시지에서 `surfaceId`를 추출. 해당 surface가 맵에 없는 경우에만:
   - `beginRendering["root"]`가 `JsonPrimitive`이면 그 `content`를 `rootId`로 사용.
   - `JsonObject`이면 `["id"]?.jsonPrimitive?.content ?: "root"`를 사용.
   - 그 외이면 `"root"` 사용.
2. `"createSurface"` 메시지에서 `surfaceId`를 추출하고 `putIfAbsent(msgSurfaceId, "root")`로 등록 (이미 있으면 덮어쓰지 않음).
3. 완성된 맵을 반환한다.

### private class A2uiTopologyValidator

`A2uiValidator` 내부에 중첩된 `private class`. 컴포넌트 배열의 위상 무결성을 검증한다.

#### 생성자 매개변수
- `catalog: A2uiCatalog`
- `rootId: String?` — 탐색 시작점. null이면 전체 사이클 검사만 수행.

#### fun validate(components: JsonArray)

1. `extractComponentRefFields()`로 참조 필드 맵을 획득.
2. `validateComponentIntegrity(components, refFieldsMap)` 호출.
3. `validateTopology(components, refFieldsMap)` 호출.

#### private fun extractComponentRefFields(): Map<String, Pair<Set<String>, Set<String>>>

`TopologyAnalyzer.extractComponentRefFields(catalog)` 위임 호출.

#### private fun validateComponentIntegrity(components: JsonArray, refFieldsMap: Map<String, Pair<Set<String>, Set<String>>>)

동작 단계:
1. 모든 컴포넌트의 `"id"` 값을 수집하여 중복 ID가 있으면 `IllegalArgumentException("Duplicate component ID: $compId")`.
2. `rootId != null`이고 `rootId`가 ID 집합에 없으면 `IllegalArgumentException("Missing root component: No component has id='$rootId'")`.
3. 각 컴포넌트의 `SchemaInspector.getComponentReferences(comp, refFieldsMap)`로 `(refId, fieldName)` 쌍을 순회. `refId`가 ID 집합에 없으면 `IllegalArgumentException` 던짐.

#### private fun validateTopology(components: JsonArray, refFieldsMap: Map<String, Pair<Set<String>, Set<String>>>)

동작 단계:
1. `SchemaInspector.updateAdjacencyList`로 인접 리스트와 전체 ID 집합 구성.
2. `rootId != null`이면: `SchemaInspector.visit(rootId, adjList)`로 방문 집합 계산. 고아(`allIds - visited`)가 있으면 첫 번째 고아에 대해 `IllegalArgumentException`.
3. `rootId == null`이면: 정렬된 `allIds`를 순회하며 방문하지 않은 노드를 DFS로 순회하여 사이클만 검사.

### private class A2uiRecursionValidator

`A2uiValidator` 내부에 중첩된 `private class`. JSON 페이로드의 전역 재귀 깊이와 함수 호출 깊이를 제한한다.

#### 생성자 매개변수
- `strictIntegrity: Boolean = true`

#### fun validate(data: JsonElement)

`traverse(data, 0, 0)` 호출.

#### private fun traverse(item: JsonElement, globalDepth: Int, funcDepth: Int)

동작 단계:
1. `globalDepth > MAX_GLOBAL_DEPTH`(50)이면 `IllegalArgumentException("Global recursion limit exceeded: Depth > $MAX_GLOBAL_DEPTH")`.
2. `item`이 `JsonArray`이면 각 원소에 `traverse(it, globalDepth + 1, funcDepth)` 재귀.
3. `item`이 `JsonObject`이면:
   a. `item["path"]`가 문자열 `JsonPrimitive`이면 정규식 `JSON_POINTER_PATTERN`으로 검사. 불일치 시 `IllegalArgumentException("Invalid path syntax: '${pathElem.content}'"`) 던짐.
   b. `"call" in item && "args" in item`이면 함수 호출 객체로 판단:
      - `funcDepth >= MAX_FUNC_CALL_DEPTH`(5)이면 `IllegalArgumentException("Recursion limit exceeded: functionCall depth > 5")`.
      - 각 `(k, v)` 쌍을 순회. `k == "args"`이면 `nextFuncDepth = funcDepth + 1`, 그 외이면 `funcDepth` 유지하여 재귀.
   c. 함수 호출 객체가 아니면 모든 값에 `funcDepth` 유지하며 재귀.
4. 그 외 원소 타입(`JsonPrimitive`, `JsonNull`)이면 아무 처리 없이 반환.

### companion object (private)

`A2uiValidator` 내부의 `private companion object`. 모든 상수를 정의한다.

| 상수명 | 값 | 설명 |
|--------|-----|------|
| `JSON_POINTER_PATTERN` | `Regex("^(?:(?:/(?:[^~/]|~[01])*)*|(?:[^~/]|~[01])+(?:/(?:[^~/]|~[01])*)*)$")` | JSON Pointer 형식 검증용 정규식 |
| `MAX_FUNC_CALL_DEPTH` | `5` | 함수 호출 최대 중첩 깊이 |
| `ROOT` | `"root"` | 기본 루트 컴포넌트 ID |
| `ID` | `"id"` | 컴포넌트 ID 필드명 |
| `PATH` | `"path"` | JSON Pointer 경로 필드명 |
| `FUNCTION_CALL` | `"functionCall"` | 오류 메시지용 함수 호출 레이블 |
| `CALL` | `"call"` | 함수 호출 객체 판별 키 |
| `ARGS` | `"args"` | 함수 호출 인수 키 |
| `MSG_SURFACE_UPDATE` | `"surfaceUpdate"` | 메시지 타입 키 |
| `MSG_UPDATE_COMPONENTS` | `"updateComponents"` | 메시지 타입 키 |
| `MSG_BEGIN_RENDERING` | `"beginRendering"` | 메시지 타입 키 |
| `MSG_CREATE_SURFACE` | `"createSurface"` | 메시지 타입 키 |
| `KEY_DOLLAR_SCHEMA` | `"$schema"` | JSON Schema 키 |
| `KEY_DOLLAR_ID` | `"$id"` | JSON Schema 기반 ID 키 |
| `PROP_ADDITIONAL_PROPERTIES` | `"additionalProperties"` | JSON Schema 속성 키 |
| `SCHEMA_DRAFT_2020_12` | `"https://json-schema.org/draft/2020-12/schema"` | Draft 버전 URI |
| `FILE_COMMON_TYPES` | `"common_types.json"` | 공통 타입 스키마 파일명 |

## 동작 흐름

`A2uiValidator` 인스턴스 생성 시 `buildValidator()`가 즉시 실행되어 버전별 최상위 `Schema` 객체를 초기화한다. `validate(a2uiJson)` 호출 시 메시지 배열을 정규화한 뒤 (1) JSON Schema 구조 검증, (2) `strictIntegrity`가 활성화된 경우 컴포넌트 그래프 위상 검증(`A2uiTopologyValidator`), (3) 모든 메시지에 대한 재귀 깊이 검증(`A2uiRecursionValidator`) 세 단계를 순서대로 수행한다. VERSION_0_9에서는 메시지 타입별 분기를 가진 커스텀 검증 로직(`validate0_9Custom`)으로 구조 검증을 대체한다.
