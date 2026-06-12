# agent_sdks/kotlin/src/test/kotlin/com/google/a2ui/conformance/A2aConformanceTest.kt

## 개요

A2A 통합 명세 적합성 테스트 클래스이다. `suites/a2a_integration.yaml` 파일에 정의된 테스트 케이스를 동적으로 로드하여 JUnit 5 `@TestFactory`로 실행한다. `A2uiA2a`와 `A2aHandler`의 API가 명세와 일치하는지를 검증하며, MockK 라이브러리로 ADK Runner/Session 의존성을 목(mock) 처리한다.

## 의존성

### 외부 패키지
- `com.fasterxml.jackson.databind.ObjectMapper` / `YAMLFactory` — YAML 파일 파싱
- `com.google.adk.agents.RunConfig`, `com.google.adk.events.Event` — ADK 에이전트 실행 이벤트
- `com.google.adk.runner.Runner` — ADK 실행기 인터페이스
- `com.google.adk.sessions.BaseSessionService`, `GetSessionConfig`, `Session` — ADK 세션 관리
- `com.google.genai.types.Content`, `Part` — GenAI 콘텐츠 타입
- `io.a2a.spec.DataPart` — A2A 데이터 파트 타입
- `io.mockk.*` — 목 생성 및 동작 정의
- `io.reactivex.rxjava3.core.Flowable`, `Maybe` — 비동기 스트림
- `kotlin.test.assertEquals`, `assertTrue` — 단언 함수
- `kotlinx.serialization.json.*` — JSON 요소 타입
- `org.junit.jupiter.api.DynamicTest`, `TestFactory` — JUnit 5 동적 테스트

### 저장소 내부 모듈
- `com.google.a2ui.a2a.A2aHandler` — A2A RPC 요청 처리기
- `com.google.a2ui.a2a.A2uiA2a` — A2UI A2A 유틸리티 (파트 생성/판별/확장 활성화/선택)
- [`ConformanceTestHelper`](ConformanceTestHelper.kt.md) — 적합성 테스트 파일 경로 및 키 상수 헬퍼

## Exports

| 이름 | 종류 |
|------|------|
| `A2aConformanceTest` | class (public, JUnit 5 테스트 클래스) |
| `A2aConformanceTest.testA2aIntegrationConformance` | `@TestFactory` 함수 |

## 테스트 케이스 명세

### testA2aIntegrationConformance(): List<DynamicTest>

`@TestFactory` 어노테이션으로 JUnit 5 동적 테스트 팩토리를 정의한다. `ConformanceTestHelper.getConformanceFile("suites/a2a_integration.yaml")`로 YAML 파일을 로드하고, `yamlMapper.readValue(..., Any::class.java)`로 역직렬화한 리스트를 순회하며 각 케이스를 `DynamicTest.dynamicTest(name) { ... }` 블록으로 변환한다. 각 케이스는 `name`, `action`, `args`, `expect` 필드를 가진 맵으로 구성된다.

#### 보조 함수: anyToJsonElement(any: Any?): JsonElement

YAML에서 역직렬화된 임의 Java 객체를 재귀적으로 `JsonElement`로 변환한다. `null → JsonNull`, `Map → JsonObject`, `List → JsonArray`, `String/Number/Boolean → JsonPrimitive`. 지원하지 않는 타입은 `IllegalArgumentException`을 던진다.

---

#### 액션: `"create_a2ui_part"`

검증 동작: `A2uiA2a.createA2uiPart(jsonElement, version)`이 `DataPart`를 반환하고, 그 메타데이터의 MIME 타입 키(`A2uiA2a.MIME_TYPE_KEY`)에 기대 값(`expect["mime_type"]`)이 들어있는지 확인한다.

픽스처: `args["data"]`를 `anyToJsonElement`로 변환한 `JsonObject`와 `args["version"]`(nullable String).

단언:
- `assertTrue(part is DataPart)`
- `assertEquals(expect["mime_type"], part.metadata?[A2uiA2a.MIME_TYPE_KEY])`

---

#### 액션: `"is_a2ui_part"`

검증 동작: `A2uiA2a.isA2uiPart(part)`가 `args["mime_type"]` 값을 메타데이터로 가진 `DataPart`에 대해 기대 Boolean을 반환하는지 확인한다.

픽스처: `mockk<DataPart>()`으로 생성된 목 `DataPart`. `part.metadata`를 `mapOf(A2uiA2a.MIME_TYPE_KEY to mimeType)`로 스텁.

단언: `assertEquals(expect as Boolean, result)`

---

#### 액션: `"try_activate_extension"`

검증 동작: `A2uiA2a.tryActivateA2uiExtension(uris, listOf("${A2uiA2a.A2UI_EXTENSION_BASE_URI}0.8")) { activated.add(it) }`를 호출하여 결과가 null이 아닌지와 활성화된 URI 목록이 기대와 일치하는지 확인한다.

픽스처: `args["uris"] as List<String>`, `activated: MutableList<String>`.

단언:
- `assertEquals(expect as Boolean, result != null)`
- `expect`가 `true`이면 `assertTrue(activated.contains("${A2uiA2a.A2UI_EXTENSION_BASE_URI}0.8"))`

---

#### 액션: `"get_extension"`

검증 동작: `A2uiA2a.getA2uiAgentExtension(version, acceptsInlineCatalogs, supportedCatalogIds)`가 반환하는 확장 객체의 `uri`와 `params`가 기대 맵과 일치하는지 확인한다.

픽스처: `args["version"]`(String), `args["accepts_inline_catalogs"]`(Boolean, 기본 false), `args["supported_catalog_ids"]`(List<String>, 기본 emptyList).

단언: `assertEquals(expect["uri"], ext.uri)`, `assertEquals(expect["params"], ext.params)`

---

#### 액션: `"try_activate"`

검증 동작: `A2uiA2a.tryActivateA2uiExtension(requested, advertised) { activated = it }`를 호출하여 반환된 버전 문자열과 활성화된 버전이 기대 맵의 `"version"` 및 `"activated"` 값과 일치하는지 확인한다.

픽스처: `args["requested"]`(List<String>), `args["advertised"]`(List<String>).

단언: `assertEquals(expect["version"], result)`, `assertEquals(expect["activated"], activated)`

---

#### 액션: `"select_newest"`

검증 동작: `A2uiA2a.selectNewestA2uiExtension(requested, advertised)`가 기대 맵의 `"newest"` 값을 반환하는지 확인한다.

픽스처: `args["requested"]`(List<String>), `args["advertised"]`(List<String>).

단언: `assertEquals(expect["newest"], result)`

---

#### 액션: `"handle_rpc"`

검증 동작: 목 `Runner`/`Session`/`Event`/`Content`/`Part` 체인을 구성하고, `A2aHandler(mockRunner).handleA2aPost(request)`가 반환하는 응답 파트 목록이 기대 파트 목록과 일치하는지 확인한다.

픽스처 및 모킹:
- `mockRunner`: `appName()` → `"test-app"`, `sessionService()` → `mockSessionService`.
- `mockSessionService.getSession(...)` → `Maybe.just(mockSession)`.
- `mockPart`: `text()` → `Optional.of(runnerOutput)`, `functionCall()` → `Optional.empty()`.
- `mockContent.parts()` → `Optional.of(listOf(mockPart))`.
- `mockEvent`: `id()` → `"test-event-id"`, `content()` → `Optional.of(mockContent)`.
- `mockRunner.runAsync(...)` → `Flowable.just(mockEvent)`.

단언:
- `assertEquals(expectParts.size, parts.size)`
- 각 파트에 대해 `kind`, `text`(있으면), `mimeType`(metadata 통해, 있으면), `data`(있으면) 비교.

---

#### 그 외 액션

`assert(false, { "Unknown action: $action" })`으로 테스트를 강제 실패시킨다.

## 동작 흐름

1. `testA2aIntegrationConformance()` 실행 시 YAML 파일을 읽어 케이스 목록을 파싱한다.
2. 각 케이스에 대해 `DynamicTest` 객체를 생성하며, JUnit 5가 이를 독립 테스트로 실행한다.
3. 각 테스트 내부에서 `action` 값에 따라 분기하여 `A2uiA2a` 또는 `A2aHandler` API를 호출하고 단언문으로 결과를 검증한다.
4. 모킹이 필요한 액션(`is_a2ui_part`, `handle_rpc`)은 MockK로 의존성을 대체한다.
