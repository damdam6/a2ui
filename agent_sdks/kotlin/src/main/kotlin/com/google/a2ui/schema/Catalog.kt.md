# agent_sdks/kotlin/src/main/kotlin/com/google/a2ui/schema/Catalog.kt

## 개요

A2UI 카탈로그의 설정(`CatalogConfig`)과 처리된 카탈로그 표현(`A2uiCatalog`)을 정의하는 핵심 파일이다. `A2uiCatalog`는 서버→클라이언트 스키마, 공통 타입 스키마, 카탈로그 스키마를 결합하여 보유하며, 컴포넌트·메시지의 정리(pruning), LLM 지시문 렌더링, 예제 파일 로딩 및 검증 기능을 제공한다. 파일 최상단에 `@file:JvmName("CatalogApi")`가 선언되어 Java에서 `CatalogApi` 클래스명으로 접근된다.

## 의존성

### 외부 패키지
- `java.io.File`, `java.net.URI`, `java.nio.file.*` — 파일시스템 탐색
- `java.util.logging.Logger` — 경고 로깅
- `kotlinx.serialization.json.*` — JSON 직렬화/역직렬화

### 저장소 내부 모듈
- [`A2uiConstants.kt`](A2uiConstants.kt.md) — 스키마 키 상수 및 LLM 블록 마커
- [`A2uiSchemaManager.kt`](A2uiSchemaManager.kt.md) — `A2uiVersion` 열거형
- [`CatalogProvider.kt`](CatalogProvider.kt.md) — `A2uiCatalogProvider` 인터페이스
- `A2uiValidator` (동일 패키지) — 유효성 검사기 (lazy 생성)

## Exports

| 이름 | 종류 |
|---|---|
| `CatalogConfig` | data class |
| `A2uiCatalog` | data class |
| `resolveExamplesPath` | internal 함수 |

## 상세 명세

### data class `CatalogConfig`

```
data class CatalogConfig(
  @JvmField val name: String,
  @JvmField val provider: A2uiCatalogProvider,
  @JvmField val examplesPath: String? = null,
  @JvmField val customCuttableKeys: Set<String>? = null,
)
```

카탈로그 하나의 설정을 담는 데이터 클래스다.

| 필드 | 타입 | 설명 |
|---|---|---|
| `name` | `String` | 카탈로그 이름 |
| `provider` | `A2uiCatalogProvider` | 스키마 로드 전략 |
| `examplesPath` | `String?` | 예제 파일 경로 (디렉토리 또는 glob 패턴, null 가능) |
| `customCuttableKeys` | `Set<String>?` | 스트리밍 커팅 가능 키 재정의 (null이면 기본값 사용) |

#### companion object 함수: `fromPath`

```
@JvmStatic @JvmOverloads
fun fromPath(
  name: String,
  catalogPath: String,
  examplesPath: String? = null,
  customCuttableKeys: Set<String>? = null,
): CatalogConfig
```

`catalogPath`를 URI로 파싱하여 scheme을 판별한다:
- `null` 또는 `"file"`: `FileSystemCatalogProvider(path)` 생성. `"file"` scheme이면 `Paths.get(uri).toString()`으로 경로 변환.
- `"http"` 또는 `"https"`: `NotImplementedError` 발생 ("HTTP support is coming soon.").
- 기타: `IllegalArgumentException` 발생.

최종적으로 `resolveExamplesPath(examplesPath)`를 적용한 경로와 함께 `CatalogConfig` 인스턴스 반환.

---

### internal fun `resolveExamplesPath(path: String?): String?`

`path`가 null이면 null 반환. URI 파싱 후 scheme을 확인한다:
- `null` 또는 `"file"`: 로컬 경로로 변환하여 반환.
- 기타 scheme: `IllegalArgumentException` 발생.

---

### data class `A2uiCatalog`

```
data class A2uiCatalog(
  @JvmField val version: A2uiVersion,
  @JvmField val name: String,
  @JvmField val serverToClientSchema: JsonObject,
  @JvmField val commonTypesSchema: JsonObject,
  @JvmField val catalogSchema: JsonObject,
  @JvmField val customCuttableKeys: Set<String>? = null,
)
```

| 필드 | 타입 | 설명 |
|---|---|---|
| `version` | `A2uiVersion` | 사양 버전 |
| `name` | `String` | 카탈로그 이름 |
| `serverToClientSchema` | `JsonObject` | 서버→클라이언트 프로토콜 스키마 |
| `commonTypesSchema` | `JsonObject` | 공통 타입 정의 스키마 |
| `catalogSchema` | `JsonObject` | 컴포넌트 정의 카탈로그 스키마 |
| `customCuttableKeys` | `Set<String>?` | 커팅 가능 키 재정의 |

#### Computed properties

- `cuttableKeys: Set<String>` — `customCuttableKeys ?: A2uiConstants.DEFAULT_CUTTABLE_KEYS`
- `validator: A2uiValidator` — `by lazy { A2uiValidator(this) }` (지연 초기화)
- `catalogId: String` — `catalogSchema["catalogId"]`를 추출. `JsonPrimitive`이고 문자열이 아니면 `require` 실패로 예외.

#### companion object

- `logger: Logger` — `Logger.getLogger(A2uiCatalog::class.java.name)`

---

### fun `withPruning(allowedComponents: List<String>?, allowedMessages: List<String>?): A2uiCatalog`

비파괴적으로 정리된 새 카탈로그 사본을 반환한다.

1. `allowedComponents`가 null이 아니면 `withPrunedComponentsInternal(allowedComponents)` 적용.
2. `allowedMessages`가 null이 아니면 `withPrunedMessages(allowedMessages)` 적용.
3. 항상 `withPrunedCommonTypes()` 적용.

---

### private fun `withPrunedComponentsInternal(allowedComponents: List<String>): A2uiCatalog`

`allowedComponents`가 비어 있으면 `this` 반환. 그렇지 않으면:

1. `catalogSchema`를 가변 맵으로 복사.
2. `"components"` 하위 `JsonObject`의 키를 `allowedComponents`에 포함된 것만 남김.
3. `"$defs"` → `"anyComponent"` 하위 `"oneOf"` 배열에서도 `pruneAnyComponentOneOf`를 통해 허용된 컴포넌트만 남김.
4. 수정된 스키마로 `copy()` 반환.

---

### private fun `withPrunedMessages(allowedMessages: List<String>): A2uiCatalog`

`allowedMessages`가 비어 있으면 `this` 반환. 버전에 따라 두 가지 경로:

- **VERSION_0_8**: `serverToClientSchema["properties"]`에서 `pruneDefsByReachability(defs=props, rootDefNames=allowedMessages, internalRefPrefix="#/properties/")` 적용.
- **기타 버전**:
  1. `"oneOf"` 배열에서 `"$ref"` 값이 `"#/$defs/<name>"` 형태이고 name이 `allowedMessages`에 있는 항목만 남김.
  2. `"$defs"`에 `pruneDefsByReachability(defs, allowedMessages, "#/$defs/")` 적용.

수정된 `serverToClientSchema`로 `copy()` 반환.

---

### private fun `withPrunedCommonTypes(): A2uiCatalog`

`commonTypesSchema["$defs"]`가 없거나 비어 있으면 `this` 반환. 그렇지 않으면:

1. `catalogSchema`와 `serverToClientSchema` 전체를 `collectRefs`로 스캔하여 외부 `$ref` 값들을 수집.
2. `"common_types.json#/$defs/"` 접두사를 가진 ref에서 정의 이름들(`rootDefs`)을 추출.
3. `pruneDefsByReachability(defs, rootDefs)`로 도달 가능한 공통 타입 정의만 남김.
4. 수정된 `commonTypesSchema`로 `copy()` 반환.

---

### private fun `collectRefs(rootElement: JsonElement, refs: MutableSet<String>)`

스택 기반 반복 DFS로 JSON 구조 전체를 순회한다. `JsonObject`에서 키가 `"$ref"`이고 값이 문자열 `JsonPrimitive`인 경우 `refs`에 추가한다. `JsonArray`의 각 항목은 스택에 추가한다. 다른 타입은 무시한다.

---

### private fun `pruneDefsByReachability(defs: JsonObject, rootDefNames: List<String>, internalRefPrefix: String = "#/$defs/"): JsonObject`

너비 우선 탐색으로 도달 가능한 정의만 남긴다.

1. `visitedDefs = mutableSetOf()`, `queue = ArrayDeque(rootDefNames)` 초기화.
2. 큐에서 이름을 꺼내 `defs`에 있고 아직 방문하지 않은 경우:
   - `visitedDefs`에 추가.
   - 해당 정의 요소에서 `collectRefs`로 내부 참조를 수집.
   - `internalRefPrefix`로 시작하는 참조에서 이름을 추출하여 큐에 추가.
3. `defs.filterKeys { it in visitedDefs }`를 `JsonObject`로 반환.

---

### private fun `pruneAnyComponentOneOf(anyCompElement: JsonObject, allowedComponents: List<String>): JsonObject`

`anyCompElement["oneOf"]`를 `JsonArray`로 추출. 각 항목의 `"$ref"` 값이 `"#/components/<name>"` 형태이면 `name`이 `allowedComponents`에 있을 때만 유지. `"#/components/"`로 시작하지 않는 참조는 항상 유지. 필터링된 배열로 `anyCompElement`를 복사하여 반환.

---

### fun `renderAsLlmInstructions(): String`

LLM 시스템 프롬프트에 삽입할 스키마 블록을 생성한다.

1. `A2uiConstants.A2UI_SCHEMA_BLOCK_START` 줄 추가.
2. `"### Server To Client Schema:"` 후 `serverToClientSchema` JSON 직렬화.
3. `commonTypesSchema["$defs"]`가 비어 있지 않으면 `"### Common Types Schema:"` 후 직렬화.
4. `"### Catalog Schema:"` 후 `catalogSchema` 직렬화.
5. `A2uiConstants.A2UI_SCHEMA_BLOCK_END`로 마무리.

---

### fun `loadExamples(path: String?, validate: Boolean = false): String`

경로가 null 또는 비어 있으면 빈 문자열 반환. 경로가 디렉토리이면 `*.json` glob 패턴으로 변환. 와일드카드 이전 부분에서 베이스 디렉토리를 추출한다.

파일 탐색:
- `FileSystems.getDefault().getPathMatcher("glob:<pattern>")` 생성.
- `/**/`를 `/`로, `^**/`를 제거한 대체 패턴도 생성하여 globstar 제로 디렉토리 매칭 지원.
- `Files.walk(startPath)`로 전체 파일 순회 후 매처에 맞는 파일만 수집.
- 오류 발생 시 `logger.warning`으로 로깅.

수집된 파일이 없으면 빈 문자열 반환 (단순 경로인데 없으면 경고 로그).

있으면 파일 경로로 알파벳 정렬 후, 각 파일을 읽어 `"---BEGIN <basename>---\n<content>\n---END <basename>---"` 형태로 변환. `validate == true`이면 `validateExample` 호출. 결과를 `"\n\n"`으로 연결하여 반환.

---

### private fun `validateExample(fullPath: String, content: String)`

`Json.parseToJsonElement(content)`로 파싱 후 `validator.validate(element)` 호출. 예외 발생 시 `IllegalArgumentException`으로 래핑하여 re-throw.

## 동작 흐름

`CatalogConfig`는 설정 단계에서 `A2uiSchemaManager`에게 전달된다. `A2uiSchemaManager`의 `init`이 각 `CatalogConfig.provider.load()`를 호출하여 원시 스키마를 가져오고, 이를 `A2uiCatalog`로 조립한다. 이후 요청마다 `withPruning`이 호출되어 불필요한 컴포넌트·메시지·공통 타입을 제거한 가벼운 사본이 만들어지며, 이 사본의 `renderAsLlmInstructions()`와 `loadExamples()`가 시스템 프롬프트 구성에 사용된다.
