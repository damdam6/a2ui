# agent_sdks/kotlin/src/main/kotlin/com/google/a2ui/a2a/A2aHandler.kt

## 개요

ADK(Agent Development Kit) 기반 에이전트를 위한 A2A(Agent-to-Agent) 프로토콜 요청 처리기다. 무거운 서버 의존성 없이 에이전트 카드 GET 요청과 JSON-RPC POST 요청을 처리할 수 있도록 단순화된 핸들러를 제공한다. LLM 이벤트에서 A2UI JSON 데이터를 추출하고 A2A Part 형태로 변환하는 책임도 담당한다.

## 의존성

### 외부 패키지
- `com.google.adk.agents.RunConfig`
- `com.google.adk.events.Event`
- `com.google.adk.runner.Runner`
- `com.google.adk.sessions.Session`
- `com.google.adk.sessions.SessionKey`
- `com.google.genai.types.Content`
- `com.google.genai.types.Part`
- `java.util.Optional`, `java.util.UUID`, `java.util.logging.Logger`
- `kotlinx.serialization.json.*`

### 저장소 내부 모듈
- [`com.google.a2ui.parser.hasA2uiParts`, `com.google.a2ui.parser.parseResponseToParts`](../parser/Parser.kt.md)
- `com.google.a2ui.schema.A2uiConstants` (schema 패키지, 별도 파일)
- [`com.google.a2ui.a2a.A2uiA2a`](A2uiA2a.kt.md)

## Exports

- `A2aHandler` (class)

## 상세 명세

### `class A2aHandler(private val runner: Runner)`

ADK `Runner` 인스턴스를 생성자로 받아 보관한다. `Runner`는 에이전트 실행, 세션 서비스, 앱 이름 조회 등에 사용된다.

---

#### `fun handleAgentCardGet(agentName: String, serverUrl: String, supportedCatalogIds: List<String> = emptyList(), version: String = A2uiConstants.VERSION_0_9): Map<String, Any>`

`/.well-known/agent-card.json` GET 요청에 응답할 맵을 생성한다.

반환 맵 구조:
- `"name"` → `agentName`
- `"url"` → `serverUrl`
- `"endpoints"` → `{ "chat": serverUrl }`
- `"capabilities"` → streaming 지원 true, extensions 목록 포함
  - extensions 항목: `uri`는 `A2uiA2a.A2UI_EXTENSION_BASE_URI + version`, `params`는 `{ "supportedCatalogIds": supportedCatalogIds }`

---

#### `@JvmOverloads fun handleA2aPost(requestBody: Map<String, Any>, sessionPreparer: ((Session, Map<*, *>) -> Unit)? = null): Map<String, Any>`

JSON-RPC 형식의 A2A POST 요청을 처리한다. 응답 맵은 항상 `"jsonrpc": "2.0"`, `"id": <요청의 id>` 를 포함하며, 처리 결과에 따라 `"result"` 또는 `"error"` 키가 추가된다.

처리 흐름:
1. `method` 필드 확인
   - `"a2a.agent.card.get"` → `handleAgentCardGet(runner.appName(), "/a2a")` 호출 후 결과를 `"result"`로 설정
   - `"a2a.agent.invoke"` 또는 `"message/send"` → 아래 처리
   - 그 외 → `{ "code": -32601, "message": "Method not found" }` 에러
2. `params.message` 맵이 null이면 `{ "code": -32602, "message": "Invalid params" }` 에러 반환
3. `contextId`를 메시지에서 추출 (없으면 `"default-context"`)
4. `sessionId = contextId`, `userId = "a2a-user"`로 세션 조회/생성
5. `sessionPreparer?.invoke(session, requestBody)` 콜백 호출
6. `runner.runAsync(...)` 로 에이전트 실행 후 이벤트 리스트 수집
7. 이벤트를 A2A Part 목록으로 변환 후 최종 메시지 맵을 `"result"`로 설정
8. 예외 발생 시 `{ "code": -32000, "message": <에러 메시지> }` 에러

---

#### `private fun extractUserContent(messageMap: Map<*, *>): Content`

A2A 메시지 맵의 `parts` 목록에서 `kind == "text"` 인 파트의 `text` 값을 모두 이어 붙여 단일 `Content` 객체(role: `"user"`)를 반환한다.

---

#### `private fun getOrCreateSession(userId: String, sessionId: String): Session`

`runner.sessionService().getSession(...)` 으로 기존 세션을 조회하고, null이면 `createSession(SessionKey(...))` 으로 새 세션을 생성한다. 두 호출 모두 `blockingGet()`으로 동기 처리한다.

---

#### `private fun translateEventsToA2aParts(events: List<*>): List<Map<String, Any>>`

`events`를 `Event` 타입으로 필터링하고, 각 이벤트의 `content().parts()` 를 순회하며 `processPart(part)`를 호출해 A2A Part 맵 목록을 평탄화(flatMap)하여 반환한다.

---

#### `private fun processPart(part: Part): List<Map<String, Any>>`

Part 하나를 처리하여 0개 이상의 A2A Part 맵을 반환한다. 처리 단계:

1. `functionCall`이 존재하고 이름이 `"send_a2ui_json_to_client"` 이면 인자 맵의 `"a2ui_json"` 값을 꺼내 `processA2uiJsonFunctionArg`로 처리
2. Part의 텍스트를 trim한 후 비어 있으면 이미 수집된 파트만 반환
3. 텍스트에 A2UI 태그가 있으면 `processA2uiTextParts`로 처리, 없으면 `{ "kind": "text", "text": <텍스트> }` 추가

---

#### `private fun processA2uiJsonFunctionArg(a2uiJsonStr: String, parsedParts: MutableList<Map<String, Any>>)`

JSON 문자열을 파싱하여 Map 또는 List이면 `{ "kind": "data", "metadata": { mimeType: A2UI_MIME_TYPE }, "data": <변환된 값> }` 형태의 파트를 추가한다. 파싱 예외는 로깅 후 무시한다.

---

#### `private fun processA2uiTextParts(text: String, parsedParts: MutableList<Map<String, Any>>)`

`parseResponseToParts(text)`로 텍스트를 파싱한다. 각 `ResponsePart`에 대해:
- 텍스트가 비어있지 않으면 `{ "kind": "text", "text": <trim된 텍스트> }` 추가
- `a2uiJson` 목록의 각 `JsonElement`를 `jsonElementToAny`로 변환하여 `{ "kind": "data", ... }` 파트 추가

예외 발생 시 원본 텍스트를 `"text"` 파트로 폴백한다.

---

#### `private fun createFinalMessage(contextId: String, events: List<*>, allParts: List<Map<String, Any>>): Map<String, Any>`

이벤트 목록의 마지막 `Event` ID(없으면 `UUID.randomUUID()`)를 `messageId`로 사용하여 최종 응답 메시지 맵을 반환한다. 구조: `{ "messageId", "contextId", "role": "model", "kind": "message", "parts": allParts }`.

---

#### `private fun jsonElementToAny(element: JsonElement): Any?`

`JsonElement`를 재귀적으로 일반 Kotlin/Java 타입으로 변환한다.
- `JsonObject` → 값을 재귀 변환한 `Map`
- `JsonArray` → 각 원소를 재귀 변환한 `List`
- `JsonPrimitive` → 문자열이면 `String`, 불리언이면 `Boolean`, 정수면 `Long`, 실수면 `Double`, 그 외 `null`

---

### Companion Object

| 심볼 | 타입 | 값 |
|---|---|---|
| `logger` | `Logger` | `Logger.getLogger(A2aHandler::class.java.name)` |
| `A2A_USER_ID` | `String` | `"a2a-user"` |
| `DEFAULT_CONTEXT_ID` | `String` | `"default-context"` |

## 동작 흐름

HTTP 서버가 `handleAgentCardGet` 또는 `handleA2aPost`를 호출한다. POST 처리 시 JSON-RPC 메서드를 분기하고, 에이전트 실행 이벤트를 동기적으로 수집한 뒤, 각 이벤트의 파트를 A2A Part 맵으로 변환하여 최종 JSON-RPC 응답 맵을 반환한다. A2UI 데이터는 함수 호출 인자와 텍스트 태그 두 경로로 감지·추출된다.
