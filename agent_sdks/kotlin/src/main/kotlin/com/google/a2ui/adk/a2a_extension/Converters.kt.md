# agent_sdks/kotlin/src/main/kotlin/com/google/a2ui/adk/a2a_extension/Converters.kt

## 개요

ADK GenAI 이벤트/파트를 A2A 프로토콜의 `Event`와 `Part` 형식으로 변환하는 두 클래스를 제공한다. `A2uiPartConverter`는 단일 GenAI `Part`를 A2UI 인식 방식으로 변환하고, `A2uiEventConverter`는 전체 ADK `Event`를 처리하여 오류와 콘텐츠를 각각 `TaskStatusUpdateEvent` 형태로 변환한다. A2UI 카탈로그 정보를 변환 과정에 자동으로 주입하는 것이 핵심 역할이다.

## 의존성

### 외부 패키지
- `com.google.adk.a2a.converters.EventConverter`
- `com.google.adk.agents.InvocationContext`
- `com.google.adk.events.Event`
- `com.google.genai.types.Content`, `com.google.genai.types.Part`
- `io.a2a.spec.Event as A2aEvent`, `io.a2a.spec.Message`, `io.a2a.spec.Message.Role.ROLE_AGENT`
- `io.a2a.spec.Part as A2aPart`, `io.a2a.spec.TaskState`, `io.a2a.spec.TaskStatus`
- `io.a2a.spec.TaskStatusUpdateEvent`, `io.a2a.spec.TextPart`
- `java.time.OffsetDateTime`, `java.util.Optional`, `java.util.UUID.randomUUID`
- `java.util.logging.Logger`
- `kotlinx.serialization.json.JsonElement`

### 저장소 내부 모듈
- [`com.google.a2ui.a2a.A2uiA2a`](../../a2a/A2uiA2a.kt.md)
- [`com.google.a2ui.parser.hasA2uiParts`, `com.google.a2ui.parser.parseResponseToParts`](../../parser/Parser.kt.md)
- `com.google.a2ui.schema.A2uiCatalog` (schema 패키지)
- [`com.google.a2ui.adk.a2a_extension.SendA2uiToClientToolset`](SendA2uiToClientToolset.kt.md)

## Exports

- `A2uiPartConverter` (class)
- `A2uiEventConverter` (class)

## 상세 명세

### `class A2uiPartConverter(private val catalog: A2uiCatalog, private val bypassToolCheck: Boolean = false)`

카탈로그를 인식하는 GenAI-to-A2A Part 변환기다.

#### `fun convert(part: Part): List<A2aPart<*>>`

단일 GenAI `Part`를 0개 이상의 A2A `Part` 목록으로 변환한다. 처리 흐름:

1. `part.functionResponse()`를 확인하여 이름이 `SendA2uiToClientToolset.TOOL_NAME`(`"send_a2ui_json_to_client"`)이거나 `bypassToolCheck`가 true인 경우:
   - `functionResponse`가 null이면 빈 리스트 반환
   - 응답 맵에 `TOOL_ERROR_KEY`가 있으면 경고 로그 후 빈 리스트 반환
   - 응답 맵의 `VALIDATED_A2UI_JSON_KEY`에서 `JsonElement`를 추출하여 `A2uiA2a.createA2uiPart(it)`로 단일 파트 리스트 반환
2. `part.functionCall()`의 이름이 `TOOL_NAME`이면 빈 리스트 반환 (함수 호출 자체는 변환하지 않음)
3. `part.text()`가 null이면 빈 리스트 반환
4. 텍스트에 A2UI 태그가 있으면 `parseResponseToParts(text, catalog.validator)`로 파싱 후 각 `ResponsePart`의 `a2uiJson` 목록을 `A2uiA2a.createA2uiPart`로 변환하여 반환
5. A2UI 태그가 없으면 빈 리스트 반환

참고: ADK Java SDK의 `PartConverter` 공개 API 부재로 인해 표준 변환은 생략되어 있으며, 클라이언트 애플리케이션이 이 로직을 맞춤 조정해야 한다는 주석이 포함되어 있다.

---

### `class A2uiEventConverter(private val catalogKey: String = "system:a2ui_catalog", private val bypassToolCheck: Boolean = false)`

세션 상태에서 카탈로그를 자동으로 조회하여 A2UI 파트 변환을 수행하는 이벤트 변환기다.

#### `fun convert(event: Event, invocationContext: InvocationContext, taskId: String? = null, contextId: String? = null): List<A2aEvent>`

ADK `Event` 하나를 0개 이상의 A2A `Event` 목록으로 변환한다. 처리 흐름:

1. `invocationContext.session().state()[catalogKey]`에서 `A2uiCatalog`를 조회한다. 없으면 빈 리스트를 반환한다.
2. `A2uiPartConverter(catalog, bypassToolCheck)` 인스턴스를 생성한다.
3. **오류 처리**: `event.errorCode()`가 존재하면 `errorMessage`를 담은 `TextPart`로 `Message`를 구성하고, `TaskState.TASK_STATE_FAILED` 상태의 `TaskStatusUpdateEvent`를 events에 추가한다.
4. **콘텐츠 처리**: `event.content()`가 존재하면:
   - `content.parts()`를 순회하며 각 파트를 `converter.convert(part)` 로 변환한다.
   - 변환 결과가 비어있으면 ADK의 `EventConverter.contentToParts(Optional.of(singleContent), false)` 표준 변환으로 폴백하여 outputParts에 추가한다.
   - `outputParts`가 비어있지 않으면 `TaskState.TASK_STATE_WORKING` 상태의 `TaskStatusUpdateEvent`를 events에 추가한다.
5. 수집된 events 목록을 반환한다.

`taskId`와 `contextId`가 null이면 ADK의 `EventConverter.taskId(event)`, `EventConverter.contextId(event)` 헬퍼로 값을 가져온다. 각 메시지의 `messageId`는 `randomUUID().toString()`, 타임스탬프는 `OffsetDateTime.now()`를 사용한다.

## 동작 흐름

ADK 에이전트 실행 결과 이벤트를 A2A 스트리밍 응답으로 변환할 때 사용된다. `A2uiEventConverter.convert`가 각 이벤트를 받아 오류와 콘텐츠를 별도의 `TaskStatusUpdateEvent`로 분리하고, 콘텐츠 내 파트는 `A2uiPartConverter.convert`를 통해 A2UI 인식 방식으로 변환한 뒤 표준 변환으로 폴백하는 구조다.
