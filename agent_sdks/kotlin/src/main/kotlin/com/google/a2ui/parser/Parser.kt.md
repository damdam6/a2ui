# agent_sdks/kotlin/src/main/kotlin/com/google/a2ui/parser/Parser.kt

## 개요

LLM 응답 텍스트에서 A2UI JSON 블록을 추출·파싱하는 파서 유틸리티 파일이다. A2UI 구분자 태그로 감싸진 JSON 블록을 정규식으로 탐지하고, 각 블록을 `PayloadFixer`를 통해 파싱·정규화한 뒤 선택적으로 스키마 검증을 수행한다. 결과는 텍스트와 JSON 데이터가 혼합된 `ResponsePart` 목록으로 반환된다.

## 의존성

### 외부 패키지
- `java.util.logging.Logger`
- `kotlinx.serialization.json.JsonElement`

### 저장소 내부 모듈
- `com.google.a2ui.schema.A2uiConstants` (schema 패키지 — `A2UI_OPEN_TAG`, `A2UI_CLOSE_TAG` 사용)
- `com.google.a2ui.schema.A2uiValidator` (schema 패키지)
- [`com.google.a2ui.parser.PayloadFixer`](PayloadFixer.kt.md)

## Exports

- `hasA2uiParts` (fun, public)
- `ResponsePart` (data class, public)
- `parseResponseToParts` (fun, public)
- `sanitizeJsonString` (fun, public)
- `A2UI_BLOCK_REGEX` (internal `Regex`)

## 상세 명세

### `private val logger`

`Logger.getLogger("com.google.a2ui.parser.Parser")` — 파일 수준 패키지 로거.

---

### `internal val A2UI_BLOCK_REGEX: Regex`

`A2uiConstants.A2UI_OPEN_TAG`와 `A2uiConstants.A2UI_CLOSE_TAG` 상수를 결합하여 구성된 정규식이다. 패턴: `<오픈태그>(.*?)<클로즈태그>`, 옵션: `DOT_MATCHES_ALL`(개행 포함 매칭). 캡처 그룹 1번이 태그 사이의 JSON 내용이다.

---

### `fun hasA2uiParts(text: String): Boolean`

텍스트에 `A2uiConstants.A2UI_OPEN_TAG` 와 `A2uiConstants.A2UI_CLOSE_TAG` 가 모두 포함되어 있으면 `true`를 반환한다. 빠른 사전 필터로 사용된다.

---

### `data class ResponsePart(val text: String, val a2uiJson: List<JsonElement>? = null)`

LLM 응답의 한 세그먼트를 나타낸다.
- `text`: 이 세그먼트의 일반 텍스트 부분 (A2UI 블록 앞의 텍스트 또는 후행 텍스트)
- `a2uiJson`: 이 세그먼트에 연결된 A2UI JSON 요소 목록. 순수 텍스트 세그먼트에서는 `null`

---

### `fun parseResponseToParts(text: String, validator: A2uiValidator? = null): List<ResponsePart>`

텍스트 전체를 파싱하여 `ResponsePart` 목록을 반환한다. 처리 단계:

1. `A2UI_BLOCK_REGEX.findAll(text)`로 모든 매치를 수집한다. 매치가 없으면 `IllegalArgumentException`을 던진다 (A2UI 태그가 없다는 메시지 포함).
2. 각 매치에 대해:
   - 매치 시작 직전까지의 텍스트를 trim하여 `textPart`로 추출한다 (`lastEnd`부터 `start`까지).
   - 캡처 그룹 1번을 `sanitizeJsonString`으로 정규화한다. 결과가 비면 `IllegalArgumentException`.
   - `PayloadFixer.parseAndFix(jsonStringCleaned)`로 `List<JsonElement>` 획득.
   - `validator?.validate(it)` 를 각 원소에 호출 (null이면 건너뜀).
   - `ResponsePart(text = textPart, a2uiJson = elements)`를 결과 목록에 추가.
   - `lastEnd`를 매치 종료 위치 다음으로 갱신.
3. 마지막 매치 이후 남은 텍스트를 trim하여 비어있지 않으면 `ResponsePart(text = trailingText, a2uiJson = null)`로 추가.
4. 완성된 `responseParts` 목록 반환.

---

### `fun sanitizeJsonString(jsonString: String): String`

LLM 출력에서 마크다운 코드 블록 표시를 제거한다. `trim()` 후 앞의 ` ```json ` 또는 ` ``` ` 접두사를 제거하고, 뒤의 ` ``` ` 접미사를 제거한 뒤 다시 `trim()`하여 반환한다.

## 동작 흐름

`hasA2uiParts`로 A2UI 태그 존재 여부를 먼저 확인한 뒤, `parseResponseToParts`로 전체 텍스트를 파싱한다. 정규식이 텍스트를 순서대로 분할하여 각 A2UI 블록과 그 앞의 텍스트를 `ResponsePart`로 묶고, 마지막 블록 이후의 후행 텍스트도 별도 `ResponsePart`로 추가한다. `PayloadFixer`가 JSON 정규화를 담당하고, `A2uiValidator`가 선택적으로 스키마 검증을 수행한다.
