# agent_sdks/kotlin/src/test/kotlin/com/google/a2ui/parser/StreamingParserTest.kt

## 개요

`StreamingParser` 및 그 버전별 구현(`StreamingParserV08`, `StreamingParserV09`)의 단위 테스트 클래스다. 메시지 타입 중복 제거, 버전별 상대/절대 경로 처리 동작, JSON 버퍼 크기 제한을 검증한다. 외부 YAML 파일 없이 인라인 픽스처 문자열만 사용하는 독립형 테스트다.

## 의존성

### 외부 패키지
- `kotlin.test.*` (Test, assertEquals, assertFailsWith, assertNotNull, assertTrue)
- `kotlinx.serialization.json.JsonArray`, `JsonObject`, `jsonPrimitive`

### 저장소 내부 모듈
- `com.google.a2ui.schema.A2uiConstants` — `A2UI_OPEN_TAG`, `A2UI_CLOSE_TAG` 상수
- `com.google.a2ui.parser.StreamingParser` — 파서 팩토리 및 공통 인터페이스
- `com.google.a2ui.parser.StreamingParserV08` — v0.8 구현체 (직접 인스턴스화)
- `com.google.a2ui.parser.StreamingParserV09` — v0.9 구현체 (직접 인스턴스화)

## Exports

- `class StreamingParserTest` — JUnit 단위 테스트 클래스

## 테스트 케이스 명세

### `deduplicatesMessageTypesCorrectly()`

**검증 동작**: `StreamingParser`가 `addMsgType` 호출 시 같은 메시지 타입을 중복 추가하지 않고, 처음 추가된 순서를 유지하는지 확인한다.

**픽스처**: `StreamingParser.create(null)`로 생성. 모킹 없음.

**절차**:
1. `addMsgType("surfaceUpdate")`를 두 번 호출 → `msgTypes`가 `["surfaceUpdate"]`임을 단언.
2. `addMsgType("beginRendering")` 호출 → `msgTypes`가 `["surfaceUpdate", "beginRendering"]`임을 단언.
3. `addMsgType("surfaceUpdate")` 재호출 → `msgTypes`가 여전히 `["surfaceUpdate", "beginRendering"]`(변경 없음)임을 단언.

---

### `sniffsAndDeduplicatesV08SurfaceUpdateMessageTypes()`

**검증 동작**: v0.8 파서가 청크 처리 중 `"surfaceUpdate"` 타입을 탐지(sniff)해 `msgTypes`에 추가하되 중복 없이 유지하고, 전체 파싱 완료 후 `msgTypes`가 비워지는지 확인한다.

**픽스처**: `StreamingParser.create(null)` (null catalog → v0.8 기본). A2UI 여는 태그와 닫는 태그로 감싼 `surfaceUpdate` JSON 페이로드를 두 청크로 분할.

**절차**:
1. `chunk1`(여는 태그 + surfaceUpdate 배열 시작 부분)을 `processChunk` → `msgTypes`에 `"surfaceUpdate"` 포함, 중복 없음 단언.
2. `chunk2`(나머지 + 닫는 태그)를 `processChunk` → `msgTypes`가 빈 리스트임을 단언 (파싱 완료로 초기화).

---

### `sniffsAndDeduplicatesV09UpdateComponentsMessageTypes()`

**검증 동작**: v0.9 파서가 `"updateComponents"` 타입을 탐지해 `msgTypes`에 추가하되 중복 없이 유지하고, 전체 파싱 완료 후 `msgTypes`가 비워지는지 확인한다.

**픽스처**: `StreamingParserV09(null)` (직접 생성; `create(null)`은 기본이 v0.8이므로). A2UI 태그로 감싼 v0.9 형식 `updateComponents` 메시지를 두 청크로 분할.

**절차**: 이전 테스트와 동일한 구조로 두 청크 처리 후 상태를 단언.

---

### `addsLeadingSlashToRelativePathsInV08()`

**검증 동작**: v0.8 파서가 컴포넌트 프로퍼티의 `"path"` 값이 상대 경로(`"some/relative/path"`)일 때 자동으로 앞에 `/`를 추가해 절대 경로(`"/some/relative/path"`)로 변환하는지 확인한다.

**픽스처**: `StreamingParserV08(null)`. `beginRendering` 청크를 먼저 처리해 surface를 초기화하고, `surfaceUpdate` 청크에 `"path": "some/relative/path"` 속성을 가진 `Text` 컴포넌트를 포함.

**검증 경로**: `parts → a2uiJson → [0]["surfaceUpdate"]["components"][0]["component"]["Text"]["text"]["path"]` 값이 `"/some/relative/path"` 인지 단언.

---

### `preservesRelativePathsInV09()`

**검증 동작**: v0.9 파서는 `"path"` 값이 상대 경로여도 수정하지 않고 그대로 유지하는지 확인한다.

**픽스처**: `StreamingParserV09(null)`. `createSurface` 청크로 surface를 초기화하고, `updateComponents` 청크에 `"path": "some/relative/path"` 속성 포함.

**검증 경로**: `parts → a2uiJson → [0]["updateComponents"]["components"][0]["text"]["path"]` 값이 `"some/relative/path"` (슬래시 미추가)인지 단언.

---

### `preservesAbsolutePathsInV09()`

**검증 동작**: v0.9 파서가 절대 경로(`"/absolute/path"`)도 수정 없이 그대로 유지하는지 확인한다.

**픽스처**: `StreamingParserV09(null)`. `createSurface` 청크 → `updateComponents` 청크 (절대 경로 포함).

**검증 경로**: 동일 경로에서 `"/absolute/path"` 값 단언.

---

### `throwsExceptionWhenJsonBufferExceedsMaxSizeLimit()`

**검증 동작**: A2UI 여는 태그 이후 JSON 버퍼에 5MB를 초과하는 공백 청크를 주입하면 `IllegalArgumentException`이 발생하는지 확인한다.

**픽스처**: `StreamingParser.create(null)`. 먼저 `processChunk(A2uiConstants.A2UI_OPEN_TAG)`로 파싱 모드 진입. 이후 `5 * 1024 * 1024 + 1` 크기 공백 문자열을 `processChunk`에 전달.

**단언**: `assertFailsWith<IllegalArgumentException>`으로 예외 발생 확인.

## 동작 흐름

각 테스트는 독립적인 파서 인스턴스를 생성해 인라인 문자열 픽스처로 `processChunk`를 직접 호출하고 결과 또는 예외를 단언한다. 외부 파일이나 모킹 프레임워크를 사용하지 않으며, `A2uiConstants`의 태그 상수로 A2UI 구분자를 구성한다.
