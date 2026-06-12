# agent_sdks/kotlin/src/main/kotlin/com/google/a2ui/schema/A2uiSchemaManager.kt

## 개요

`A2uiSchemaManager`는 `InferenceStrategy`를 구현하며, 하나 이상의 카탈로그 설정을 취합하여 LLM에게 전달할 시스템 프롬프트를 생성하는 중심 조율자다. 초기화 시 번들 리소스에서 서버→클라이언트 스키마와 공통 타입 스키마를 로드하고, 각 카탈로그 스키마에 수정자(modifier) 체인을 적용한다. 클라이언트 UI 역량(capabilities)을 입력받아 적절한 카탈로그를 선택·병합·정리(prune)하는 협상 로직을 포함한다. 또한 `A2uiVersion` 열거형을 이 파일에서 함께 정의한다.

## 의존성

### 외부 패키지
- `kotlinx.serialization.json.*` — `JsonArray`, `JsonObject`, `jsonObject`, `jsonPrimitive`

### 저장소 내부 모듈
- [`A2uiConstants.kt`](A2uiConstants.kt.md) — 스키마 키 상수 (`INLINE_CATALOGS_KEY`, `SUPPORTED_CATALOG_IDS_KEY`, `CATALOG_COMPONENTS_KEY`, `INLINE_CATALOG_NAME`, `DEFAULT_WORKFLOW_RULES`)
- [`Catalog.kt`](Catalog.kt.md) — `A2uiCatalog`, `CatalogConfig`
- `com.google.a2ui.InferenceStrategy` — 인터페이스 (구현 대상)
- `SchemaResourceLoader` (동일 패키지) — 번들 리소스 로더

## Exports

| 이름 | 종류 |
|---|---|
| `A2uiVersion` | enum class |
| `A2uiSchemaManager` | class |

## 상세 명세

### enum class `A2uiVersion`

```
enum class A2uiVersion(
  val value: String,
  val serverToClientSchemaPath: String,
  val commonTypesSchemaPath: String? = null,
)
```

A2UI 사양의 지원 버전을 열거한다.

| 엔트리 | `value` | `serverToClientSchemaPath` | `commonTypesSchemaPath` |
|---|---|---|---|
| `VERSION_0_8` | `"0.8"` | `"server_to_client.json"` | `null` |
| `VERSION_0_9` | `"0.9"` | `"server_to_client.json"` | `"common_types.json"` |
| `VERSION_0_9_1` | `"0.9.1"` | `"server_to_client.json"` | `"common_types.json"` |

---

### class `A2uiSchemaManager`

```
class A2uiSchemaManager @JvmOverloads constructor(
  private val version: A2uiVersion,
  catalogs: List<CatalogConfig> = emptyList(),
  @JvmField val acceptsInlineCatalogs: Boolean = false,
  private val schemaModifiers: List<(JsonObject) -> JsonObject> = emptyList(),
) : InferenceStrategy
```

#### 필드

| 이름 | 타입 | 가시성 | 설명 |
|---|---|---|---|
| `serverToClientSchema` | `JsonObject` | private | 번들 리소스에서 로드한 서버→클라이언트 스키마 |
| `commonTypesSchema` | `JsonObject` | private | 번들 리소스에서 로드한 공통 타입 스키마 (버전 0.8은 빈 객체) |
| `supportedCatalogs` | `MutableList<A2uiCatalog>` | private | 초기화된 카탈로그 목록 |
| `catalogExamplePaths` | `MutableMap<String, String?>` | private | catalogId → 예제 경로 매핑 |
| `supportedCatalogIds` | `List<String>` | public (computed) | `supportedCatalogs`의 catalogId 목록 |
| `acceptsInlineCatalogs` | `Boolean` | `@JvmField` public | 인라인 카탈로그 수용 여부 |

#### init 블록

1. `SchemaResourceLoader.loadFromBundledResource(version.value, version.serverToClientSchemaPath)` 호출 → 결과 없으면 빈 `JsonObject` → `applyModifiers` 적용 → `serverToClientSchema` 저장.
2. `version.commonTypesSchemaPath`가 있으면 동일 방식으로 로드·수정 → `commonTypesSchema` 저장; 없으면 빈 `JsonObject`.
3. `catalogs` 목록의 각 `CatalogConfig`에 대해:
   - `config.provider.load()`로 카탈로그 스키마 로드 후 `applyModifiers` 적용.
   - `A2uiCatalog(version, config.name, catalogSchema, serverToClientSchema, commonTypesSchema)` 생성.
   - `supportedCatalogs`에 추가, `catalogExamplePaths[catalog.catalogId] = config.examplesPath` 저장.

---

### private fun `applyModifiers(schema: JsonObject): JsonObject`

`schemaModifiers` 리스트를 `fold`로 순차 적용하여 최종 변환된 `JsonObject`를 반환한다. 각 modifier는 `(JsonObject) -> JsonObject` 함수다.

---

### private fun `selectCatalog(clientUiCapabilities: JsonObject?): A2uiCatalog`

클라이언트 역량 객체를 기반으로 사용할 카탈로그를 선택하거나 병합한다.

**처리 단계**

1. `supportedCatalogs`가 비어 있으면 `check` 실패로 예외 발생.
2. `clientUiCapabilities`가 `null`이면 `supportedCatalogs.first()` 반환.
3. `inlineCatalogs`: `clientUiCapabilities["inlineCatalogs"]`를 `JsonArray`로 추출.
4. `clientSupportedCatalogIds`: `clientUiCapabilities["supportedCatalogIds"]`를 문자열 목록으로 추출.
5. `acceptsInlineCatalogs == false`이고 `inlineCatalogs`가 비어 있지 않으면 `IllegalArgumentException` 발생.
6. **인라인 카탈로그 병합 경로** (`inlineCatalogs` 비어있지 않음):
   - 기본 카탈로그(`baseCatalog`)를 `supportedCatalogs.first()`로 초기화.
   - `clientSupportedCatalogIds`가 있으면 에이전트 지원 카탈로그와 매핑하여 첫 번째 매칭 카탈로그로 `baseCatalog` 교체.
   - `baseCatalog.catalogSchema`를 복사한 가변 맵 생성.
   - 각 인라인 카탈로그 스키마에 `applyModifiers` 적용 후 `components` 항목을 복사 맵에 병합(`putAll`).
   - 병합된 스키마로 새 `A2uiCatalog` (name=`"inline"`) 생성 후 반환.
7. `clientSupportedCatalogIds`가 비어 있으면 `supportedCatalogs.first()` 반환.
8. `clientSupportedCatalogIds`를 순회하며 에이전트 지원 카탈로그에서 첫 매칭을 찾아 반환.
9. 매칭 없으면 `IllegalArgumentException` 발생 (에이전트 지원 catalogId 목록 포함 메시지).

---

### fun `getSelectedCatalog(...): A2uiCatalog`

```
@JvmOverloads
fun getSelectedCatalog(
  clientUiCapabilities: JsonObject? = null,
  allowedComponents: List<String> = emptyList(),
  allowedMessages: List<String> = emptyList(),
): A2uiCatalog
```

`selectCatalog(clientUiCapabilities)`를 호출한 뒤, 반환된 카탈로그에 `withPruning(allowedComponents, allowedMessages)`를 적용하여 미사용 컴포넌트·메시지를 제거한 카탈로그를 반환한다.

---

### fun `loadExamples(catalog: A2uiCatalog, validate: Boolean = false): String`

```
@JvmOverloads
fun loadExamples(catalog: A2uiCatalog, validate: Boolean = false): String
```

`catalogExamplePaths[catalog.catalogId]`에서 경로를 꺼내 `catalog.loadExamples(path, validate)`를 호출한다. 경로가 없으면 빈 문자열을 반환한다.

---

### fun `generateSystemPrompt(...): String`

```
override fun generateSystemPrompt(
  roleDescription: String,
  workflowDescription: String,
  uiDescription: String,
  clientUiCapabilities: JsonObject?,
  allowedComponents: List<String>,
  allowedMessages: List<String>,
  includeSchema: Boolean,
  includeExamples: Boolean,
  validateExamples: Boolean,
): String
```

LLM에 전달할 완전한 시스템 프롬프트를 조립하여 반환한다.

**조립 단계**

1. `parts` 리스트에 `roleDescription` 추가.
2. `workflowDescription`이 비어 있으면 `A2uiConstants.DEFAULT_WORKFLOW_RULES`, 비어있지 않으면 `DEFAULT_WORKFLOW_RULES + "\n" + workflowDescription`을 워크플로우로 사용하여 `"## Workflow Description:\n<workflow>"` 섹션 추가.
3. `uiDescription`이 비어 있지 않으면 `"## UI Description:\n<uiDescription>"` 섹션 추가.
4. `getSelectedCatalog(clientUiCapabilities, allowedComponents, allowedMessages)`로 선택된 카탈로그 획득.
5. `includeSchema == true`이면 `selectedCatalog.renderAsLlmInstructions()` 결과 추가.
6. `includeExamples == true`이면 `loadExamples(selectedCatalog, validateExamples)` 결과가 비어 있지 않을 때 `"### Examples:\n<examplesStr>"` 추가.
7. `parts.joinToString("\n\n")`로 최종 문자열 반환.

## 동작 흐름

초기화 시 스키마와 카탈로그가 로드·변환되어 내부 상태에 캐시된다. 런타임에는 `generateSystemPrompt`가 호출되며, 이 메서드가 `selectCatalog`를 통해 클라이언트 역량에 맞는 카탈로그를 협상하고, `withPruning`으로 필요한 컴포넌트·메시지만 남긴 후, `renderAsLlmInstructions`와 예제 로더를 통해 완성된 시스템 프롬프트 문자열을 조립한다.
