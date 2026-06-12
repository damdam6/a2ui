# agent_sdks/kotlin/src/main/kotlin/com/google/a2ui/parser/PayloadFixer.kt

## 개요

LLM이 출력한 불완전하거나 비표준적인 JSON 문자열을 파싱 가능한 상태로 수정하는 유틸리티 싱글턴이다. 스마트 따옴표 정규화, 후행 쉼표 제거 등 두 가지 자동 수정 전략을 적용하며, 파싱 결과를 `JsonElement` 목록으로 반환한다. 단일 JSON 객체는 목록으로 감싸고, JSON 배열은 그대로 반환한다.

## 의존성

### 외부 패키지
- `java.util.logging.Logger`
- `kotlinx.serialization.json.Json`
- `kotlinx.serialization.json.JsonArray`
- `kotlinx.serialization.json.JsonElement`
- `kotlinx.serialization.json.JsonObject`

### 저장소 내부 모듈
없음.

## Exports

- `PayloadFixer` (object)

## 상세 명세

### `object PayloadFixer`

#### `private val logger`

`Logger.getLogger(PayloadFixer::class.java.name)` — 내부 로거.

---

#### `fun parseAndFix(payload: String): List<JsonElement>`

최상위 진입점. 처리 흐름:

1. `normalizeSmartQuotes(payload)` 로 스마트 따옴표를 직선 따옴표로 변환한다.
2. `parse(normalizedPayload)` 를 시도한다.
3. 파싱에 실패하면 `warning` 레벨 로그를 남기고, `removeTrailingCommas(normalizedPayload)` 로 후행 쉼표를 제거한 뒤 다시 `parse(...)` 를 시도한다.
4. 두 번째 시도도 실패하면 예외가 전파된다.

---

#### `fun normalizeSmartQuotes(jsonStr: String): String`

다음 유니코드 문자를 ASCII 등가로 치환한다:

| 유니코드 | 설명 | 치환 |
|---|---|---|
| `“` | `"` (왼쪽 큰따옴표) | `"` |
| `”` | `"` (오른쪽 큰따옴표) | `"` |
| `‘` | `'` (왼쪽 작은따옴표) | `'` |
| `’` | `'` (오른쪽 작은따옴표) | `'` |

---

#### `private fun parse(payload: String): List<JsonElement>`

`Json.parseToJsonElement(payload)` 로 파싱한 뒤:
- `JsonArray`이면 `.toList()` 로 변환하여 반환
- `JsonObject`이면 `info` 로그 후 `listOf(element)` 반환
- 그 외 기본 타입이면 `IllegalArgumentException` 발생 (`"Payload must be a JSON Array or Object, got: ..."` 메시지)
- 파싱 자체가 실패하면 `severe` 로그 후 원인을 포함한 `IllegalArgumentException` 재발생

---

#### `fun removeTrailingCommas(jsonStr: String): String`

문자열을 문자 단위로 순회하며 후행 쉼표(객체 또는 배열의 마지막 원소 뒤에 오는 쉼표)를 제거한다. 따옴표 인식 방식으로 문자열 내부 콤마를 건드리지 않는다.

알고리즘 상세:
- `inString` 불리언으로 현재 따옴표 내부 여부를 추적한다. 이스케이프되지 않은 `"` 문자를 만날 때 토글된다 (이전 문자가 `\`가 아닐 때).
- 문자열 외부(`!inString`)에서:
  - `','` → `lastCommaIndex = result.length` (현재 StringBuilder 길이를 저장)
  - `']'` 또는 `'}'` 이고 `lastCommaIndex != -1` → 마지막 쉼표 이후 StringBuilder 내용이 공백뿐이면 해당 쉼표 문자를 삭제하고 `lastCommaIndex = -1`로 초기화
  - 공백이 아닌 다른 문자 → `lastCommaIndex = -1` (쉼표와 닫는 괄호 사이에 다른 내용이 있으면 후행 쉼표가 아님)
- 수정이 발생하면 `warning` 레벨 로그를 남긴다.
- 최종 수정된 문자열을 반환한다.

## 동작 흐름

`Parser.parseResponseToParts`가 `PayloadFixer.parseAndFix`를 호출한다. 먼저 스마트 따옴표를 정규화하고, 1차 파싱 성공 시 바로 결과 반환, 실패 시 후행 쉼표 제거 후 2차 파싱을 시도한다. 두 자동 수정 모두 적용해도 파싱에 실패하면 예외를 전파하여 호출자가 처리하게 한다.
