# agent_sdks/kotlin/src/test/kotlin/com/google/a2ui/conformance/AdkExtensionsConformanceTest.kt

## 개요

ADK 확장 기능 명세 적합성 테스트 클래스이다. `suites/adk_extensions.yaml` 파일에 정의된 테스트 케이스를 동적으로 로드하여 JUnit 5 `@TestFactory`로 실행한다. `A2uiEventConverter`의 이벤트 변환 동작과 `SendA2uiToClientToolset`의 툴 실행 동작이 명세에 부합하는지를 검증하며, MockK로 ADK 세션/컨텍스트/이벤트 의존성을 목(mock) 처리한다.

## 의존성

### 외부 패키지
- `com.fasterxml.jackson.databind.ObjectMapper` / `YAMLFactory` — YAML 파일 파싱
- `com.google.adk.a2a.converters.EventConverter` — A2A 이벤트 변환기 (static 목 처리 대상)
- `com.google.adk.agents.InvocationContext`, `ReadonlyContext` — ADK 에이전트 컨텍스트
- `com.google.adk.events.Event` — ADK 에이전트 이벤트
- `com.google.adk.sessions.Session` — ADK 세션
- `com.google.adk.tools.ToolContext` — 툴 실행 컨텍스트
- `com.google.common.collect.ImmutableList` — 구아바 불변 리스트
- `com.google.genai.types.Content`, `FinishReason`, `Part` — GenAI 콘텐츠 타입
- `io.a2a.spec.TaskState`, `TaskStatusUpdateEvent`, `TextPart` — A2A 태스크 상태/이벤트/파트 타입
- `io.mockk.*` — 목 생성, 스텁, 정적 목킹
- `kotlin.test.assertEquals`, `assertIs`, `assertTrue` — 단언 함수
- `kotlinx.serialization.json.Json`, `JsonObject`, `jsonObject` — JSON 파싱
- `org.junit.jupiter.api.DynamicTest`, `TestFactory` — JUnit 5 동적 테스트
- `java.util.Optional`, `java.util.concurrent.ConcurrentHashMap`

### 저장소 내부 모듈
- `com.google.a2ui.adk.a2a_extension.A2uiEventConverter` — ADK 이벤트를 A2A TaskStatusUpdateEvent로 변환하는 클래스
- `com.google.a2ui.adk.a2a_extension.SendA2uiToClientToolset` — A2UI JSON을 클라이언트에 전송하는 툴셋
- `com.google.a2ui.schema.A2uiCatalog` — 카탈로그 데이터 클래스
- `com.google.a2ui.schema.A2uiVersion` — 버전 열거형
- [`ConformanceTestHelper`](ConformanceTestHelper.kt.md) — 적합성 테스트 파일 경로 및 키 상수 헬퍼

## Exports

| 이름 | 종류 |
|------|------|
| `AdkExtensionsConformanceTest` | class (public, JUnit 5 테스트 클래스) |
| `AdkExtensionsConformanceTest.testAdkExtensionsConformance` | `@TestFactory` 함수 |

## 테스트 케이스 명세

### testAdkExtensionsConformance(): List<DynamicTest>

`@TestFactory` 어노테이션. `ConformanceTestHelper.getConformanceFile("suites/adk_extensions.yaml")`로 YAML 파일을 로드하고, 역직렬화한 리스트를 순회하며 각 케이스를 `DynamicTest.dynamicTest(name) { ... }`로 변환한다. 각 케이스는 `name`, `action`, `args`, `expect`(또는 `expect_empty`) 필드를 가진 맵이다.

---

#### 액션: `"convert_event"`

검증 동작: `A2uiEventConverter().convert(mockEvent, context)` 결과가 기대 상태/메시지와 일치하는지 확인한다.

픽스처 및 모킹:
- `session`: `state()` → `ConcurrentHashMap<String, Any>`. `args["has_catalog"]`이 `true`이면 `state["system:a2ui_catalog"] = mockk<A2uiCatalog>()`.
- `context`: `context.session()` → `session`.
- `mockEvent`:
  - `args["error_code"]`가 non-null이면 `mockEvent.errorCode()` → `Optional.of(mockk<FinishReason>())`. null이면 `Optional.empty()`.
  - `mockEvent.errorMessage()` → `Optional.ofNullable(args["error_message"])`.
  - `mockEvent.author()` → `"test_author"`.
  - `args["content_text"]`가 non-null이면:
    - `mockGenaiPart`: `functionResponse()/functionCall()` → `Optional.empty()`, `text()` → `Optional.of(contentText)`, `thought()` → `Optional.empty()`.
    - `mockContent.parts()` → `Optional.of(listOf(mockGenaiPart))`.
    - `mockEvent.content()` → `Optional.of(mockContent)`.
    - `mockkStatic(EventConverter::class)` 후 `EventConverter.contentToParts(any(), false)` → `ImmutableList.of(TextPart(contentText))`.
  - `contentText`가 null이면 `mockEvent.content()` → `Optional.empty()`.
  - `mockkStatic(EventConverter::class)` 후 `EventConverter.taskId(mockEvent)` → `"task-1"`, `EventConverter.contextId(mockEvent)` → `"context-1"`.

단언 (expect_empty가 true인 경우):
- `assertTrue(results.isEmpty())`

단언 (정상 경우):
- `assertEquals(1, results.size)`
- `assertIs<TaskStatusUpdateEvent>(result)`
- `assertEquals("task-1", result.taskId())`
- `assertEquals("context-1", result.contextId())`
- `result.status().state()`이 `"FAILED"` → `TaskState.TASK_STATE_FAILED`, `"WORKING"` → `TaskState.TASK_STATE_WORKING` (그 외 `IllegalArgumentException`)
- `result.status().message()!!.parts()!![0]`를 `TextPart`로 캐스팅하여 `text()`가 `expect["message"]`와 일치하는지 확인

---

#### 액션: `"execute_tool"`

검증 동작: `SendA2uiToClientToolset`으로 생성한 툴에 `toolArgs`를 전달하여 실행했을 때 성공/실패 여부와 결과 내용이 기대와 일치하는지 확인한다.

픽스처 및 구성:
- `args["a2ui_json"]`이 non-null이면 `toolArgs = mapOf("a2ui_json" to a2uiJsonStr)`. null이면 `args` 자체를 `Map<String, Any>`로 사용.
- 더미 `serverToClientSchema`: `{"type": "object", "properties": {"beginRendering": {"type": "object"}}}` (JSON 파싱).
- 더미 `catalogSchema`: `{"catalogId": "dummy", "components": {"TestComp": {"type": "object"}}}` (JSON 파싱).
- `dummyCatalog`: `A2uiCatalog(version = A2uiVersion.VERSION_0_8, name = "dummy", serverToClientSchema = ..., commonTypesSchema = JsonObject(emptyMap()), catalogSchema = ...)`.
- `mockContext`: `mockk<ReadonlyContext>(relaxed = true)`.
- `mockToolContext`: `mockk<ToolContext>(relaxed = true)`.
- `toolset = SendA2uiToClientToolset.create(true, dummyCatalog, "")`.
- `tool = toolset.getTools(mockContext).blockingFirst()`.
- `result = tool.runAsync(toolArgs, mockToolContext).blockingGet()`.

단언 (`expect["success"]`가 `true`인 경우):
- `assertTrue(result.containsKey(SendA2uiToClientToolset.VALIDATED_A2UI_JSON_KEY))`
- `expect["contains_validated_json"]`이 `true`이면 검증된 페이로드 문자열에 `"beginRendering"`이 포함되는지 확인

단언 (`expect["success"]`가 `false`인 경우):
- `assertTrue(result.containsKey(SendA2uiToClientToolset.TOOL_ERROR_KEY))`
- `expect["error_contains"]`가 non-null이면 오류 메시지 문자열에 해당 부분 문자열이 포함되는지 확인

---

#### 그 외 액션

`assert(false, { "Unknown action: $action" })`으로 테스트를 강제 실패시킨다.

## 동작 흐름

1. `testAdkExtensionsConformance()` 실행 시 YAML 파일을 읽어 케이스 목록을 파싱한다.
2. 각 케이스를 `DynamicTest` 객체로 변환하며 JUnit 5가 독립 테스트로 실행한다.
3. `"convert_event"` 케이스는 MockK로 ADK `Event`/`Session`/`InvocationContext` 의존성과 `EventConverter` 정적 메서드를 목 처리하여 `A2uiEventConverter`의 변환 결과를 단언한다.
4. `"execute_tool"` 케이스는 더미 `A2uiCatalog`와 목 컨텍스트를 사용하여 `SendA2uiToClientToolset`의 툴 실행 결과를 단언한다.
