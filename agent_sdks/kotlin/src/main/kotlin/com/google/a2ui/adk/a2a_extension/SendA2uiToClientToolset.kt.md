# agent_sdks/kotlin/src/main/kotlin/com/google/a2ui/adk/a2a_extension/SendA2uiToClientToolset.kt

## 개요

ADK 에이전트에 A2UI JSON 전송 기능을 제공하는 `BaseToolset` 구현체다. 동적 활성화 여부, 카탈로그 제공자, 예시 제공자를 람다로 주입받아 런타임에 에이전트가 UI 컴포넌트를 렌더링할 수 있는 도구(`send_a2ui_json_to_client`)를 노출한다. 도구 호출 시 A2UI JSON을 파싱하고 카탈로그 스키마로 검증한 뒤 결과를 반환한다.

## 의존성

### 외부 패키지
- `com.google.adk.agents.ReadonlyContext`
- `com.google.adk.models.LlmRequest`
- `com.google.adk.tools.BaseTool`, `com.google.adk.tools.BaseToolset`, `com.google.adk.tools.ToolContext`
- `com.google.genai.types.FunctionDeclaration`, `com.google.genai.types.Schema`, `com.google.genai.types.Type`
- `io.reactivex.rxjava3.core.Completable`, `io.reactivex.rxjava3.core.Flowable`, `io.reactivex.rxjava3.core.Single`
- `java.util.Optional`, `java.util.logging.Logger`

### 저장소 내부 모듈
- `com.google.a2ui.schema.A2uiCatalog` (schema 패키지)
- [`com.google.a2ui.adk.a2a_extension.A2uiPartConverter`](Converters.kt.md)

## Exports

- `A2uiEnabledProvider` (typealias)
- `A2uiCatalogProvider` (typealias)
- `A2uiExamplesProvider` (typealias)
- `SendA2uiToClientToolset` (class)

## 상세 명세

### 타입 별칭

| 이름 | 정의 |
|---|---|
| `A2uiEnabledProvider` | `(ReadonlyContext) -> Boolean` |
| `A2uiCatalogProvider` | `(ReadonlyContext) -> A2uiCatalog` |
| `A2uiExamplesProvider` | `(ReadonlyContext) -> String` |

---

### `class SendA2uiToClientToolset @JvmOverloads constructor(private val a2uiEnabled: A2uiEnabledProvider, private val a2uiCatalog: A2uiCatalogProvider, private val a2uiExamples: A2uiExamplesProvider) : BaseToolset`

#### 필드/상태
- `logger`: 클래스 로거
- `uiTools`: `listOf(SendA2uiJsonToClientTool())` — 항상 단일 도구 인스턴스를 보유

#### `override fun getTools(readonlyContext: ReadonlyContext): Flowable<BaseTool>`

`a2uiEnabled(readonlyContext)`가 true이면 `uiTools`를 `Flowable`로 방출하고, false이면 `Flowable.empty()`를 반환한다. 각 경우에 `info` 레벨 로그를 남긴다.

#### `override fun close()`

아무 동작도 하지 않는다.

#### `fun getPartConverter(ctx: ReadonlyContext): A2uiPartConverter`

`a2uiCatalog(ctx)`로 카탈로그를 조회하여 `A2uiPartConverter` 인스턴스를 생성·반환한다.

---

### `inner class SendA2uiJsonToClientTool : BaseTool(TOOL_NAME, "Sends A2UI JSON to the client to render rich UI natively. Always prefer this over returning raw JSON.")`

#### `override fun declaration(): Optional<FunctionDeclaration>`

함수 선언을 반환한다. 파라미터 스키마는 `OBJECT` 타입이며 `"a2ui_json"` 이라는 `STRING` 타입의 필수 속성 하나를 포함한다. 해당 속성의 description은 `"The A2UI JSON payload to send to the client."`.

#### `override fun processLlmRequest(llmRequestBuilder: LlmRequest.Builder, toolContext: ToolContext): Completable`

`Completable.fromAction`으로 다음을 수행한다:
1. `a2uiCatalog(toolContext)`로 카탈로그 조회
2. `catalog.renderAsLlmInstructions()`로 스키마 지시문 생성
3. `a2uiExamples(toolContext)`로 예시 문자열 가져오기
4. `llmRequestBuilder.appendInstructions(listOf(instruction, examples))` 호출
5. `info` 레벨 로그 기록

이후 `super.processLlmRequest(llmRequestBuilder, toolContext)`를 `.andThen()`으로 체이닝한다.

#### `override fun runAsync(args: Map<String, Any>, toolContext: ToolContext): Single<Map<String, Any>>`

`Single.fromCallable`로 다음을 수행한다:
1. `args["a2ui_json"]`에서 문자열 추출 — 없으면 `IllegalArgumentException` 발생
2. `a2uiCatalog(toolContext)` 호출
3. `Json.parseToJsonElement(a2uiJsonStr)` 으로 파싱
4. `catalog.validator.validate(a2uiJsonPayload)` 로 스키마 검증
5. 성공 시 `{ VALIDATED_A2UI_JSON_KEY: <JsonElement> }` 반환
6. 예외 발생 시 에러 메시지를 구성하고 `{ TOOL_ERROR_KEY: <메시지> }` 반환

---

### Companion Object

| 이름 | 타입 | 값 | 공개 여부 |
|---|---|---|---|
| `TOOL_NAME` | `String` const | `"send_a2ui_json_to_client"` | public |
| `VALIDATED_A2UI_JSON_KEY` | `String` const | `"validated_a2ui_json"` | public |
| `A2UI_JSON_ARG_NAME` | `String` const | `"a2ui_json"` | private |
| `TOOL_ERROR_KEY` | `String` const | `"error"` | public |

#### `@JvmStatic @JvmOverloads fun create(enabled: Boolean, catalog: A2uiCatalog, examples: String = ""): SendA2uiToClientToolset`

상수 값으로 `SendA2uiToClientToolset`을 생성하는 팩토리 메서드다. 세 람다를 각각 `{ enabled }`, `{ catalog }`, `{ examples }` 로 고정하여 생성자를 호출한다.

## 동작 흐름

에이전트가 LLM 요청을 준비할 때 `getTools`로 도구 목록을 받는다. A2UI가 활성화된 경우 `SendA2uiJsonToClientTool`이 포함되며, 이 도구의 `processLlmRequest`가 카탈로그 스키마와 예시를 시스템 지시문에 삽입한다. LLM이 도구를 호출하면 `runAsync`가 JSON을 파싱·검증하고 결과를 반환하며, 이 결과는 `A2uiPartConverter`나 `A2aHandler`가 후속으로 A2A Part로 변환한다.
