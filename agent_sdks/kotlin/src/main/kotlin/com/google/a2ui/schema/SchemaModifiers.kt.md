# agent_sdks/kotlin/src/main/kotlin/com/google/a2ui/schema/SchemaModifiers.kt

## 개요

A2UI JSON 스키마에 적용할 수 있는 표준 변환 함수들을 모아둔 싱글턴 객체다. 현재는 `"additionalProperties": false` 제약을 재귀적으로 제거하는 함수 하나를 제공한다. 이 변환은 LLM이 스키마에 정의되지 않은 추가 프로퍼티를 생성하더라도 유효성 검사가 실패하지 않도록 허용하고자 할 때 사용된다.

## 의존성

### 외부 패키지
- `kotlinx.serialization.json.*` — `JsonArray`, `JsonElement`, `JsonObject`, `JsonPrimitive`, `booleanOrNull`

### 저장소 내부 모듈
없음.

## Exports

| 이름 | 종류 |
|---|---|
| `SchemaModifiers` | object (싱글턴) |

## 상세 명세

### object `SchemaModifiers`

#### fun `removeStrictValidation(schema: JsonObject): JsonObject`

JSON 스키마 전체를 재귀 탐색하여 `"additionalProperties": false` 항목을 제거한다.

1. `recursiveRemoveStrict(schema)`를 호출하여 변환된 결과를 `JsonObject`로 캐스팅하여 반환한다.

---

#### private fun `recursiveRemoveStrict(element: JsonElement): JsonElement`

JSON 요소 타입에 따라 분기:

- **`JsonObject`**: 각 `(key, value)` 쌍을 필터링한다. `key == "additionalProperties"` 이고 값이 `JsonPrimitive`이며 `booleanOrNull == false`인 경우 해당 항목을 제거한다. 나머지 항목의 `value`에 대해서는 `recursiveRemoveStrict`를 재귀 호출하여 중첩 구조를 처리한다. 결과를 새 `JsonObject`로 반환.
- **`JsonArray`**: 각 요소에 `recursiveRemoveStrict`를 적용하여 새 `JsonArray`로 반환.
- **기타** (`JsonPrimitive`, `JsonNull`): 그대로 반환.

경계 케이스: `"additionalProperties"` 값이 `false`가 아닌 경우(예: `true` 또는 스키마 객체)는 제거하지 않고 유지된다.

## 동작 흐름

`A2uiSchemaManager`의 `schemaModifiers` 리스트에 `SchemaModifiers::removeStrictValidation`을 넣어 사용한다. `applyModifiers`가 스키마를 로드한 직후 이 변환을 적용하면, 서버→클라이언트 스키마, 공통 타입 스키마, 카탈로그 스키마 모두에서 strict 유효성 검사 제약이 제거된 상태로 LLM에게 전달된다.
