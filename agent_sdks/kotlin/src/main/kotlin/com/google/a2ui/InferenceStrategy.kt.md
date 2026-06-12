# agent_sdks/kotlin/src/main/kotlin/com/google/a2ui/InferenceStrategy.kt

## 개요

LLM 시스템 프롬프트 생성 전략의 계약(contract)을 정의하는 인터페이스 파일이다. 이 인터페이스를 구현하는 클래스는 에이전트 역할 설명, A2UI JSON 스키마 제약, 워크플로 가이드라인, 예시를 조합하여 완전한 시스템 프롬프트 문자열을 반환해야 한다. 클라이언트가 보고한 UI 역량이나 허용 컴포넌트 목록을 바탕으로 스키마를 선택적으로 축소할 수 있는 파라미터도 포함한다.

## 의존성

### 외부 패키지
- `kotlinx.serialization.json.JsonObject`

### 저장소 내부 모듈
없음.

## Exports

- `InferenceStrategy` (interface)

## 상세 명세

### `interface InferenceStrategy`

#### `fun generateSystemPrompt(...): String`

**시그니처:**
```
fun generateSystemPrompt(
  roleDescription: String,
  workflowDescription: String = "",
  uiDescription: String = "",
  clientUiCapabilities: JsonObject? = null,
  allowedComponents: List<String> = emptyList(),
  allowedMessages: List<String> = emptyList(),
  includeSchema: Boolean = false,
  includeExamples: Boolean = false,
  validateExamples: Boolean = false,
): String
```

각 파라미터의 의미:

| 파라미터 | 타입 | 기본값 | 역할 |
|---|---|---|---|
| `roleDescription` | `String` | 필수 | 에이전트의 기본 역할/페르소나 텍스트 |
| `workflowDescription` | `String` | `""` | 에이전트 행동을 안내하는 워크플로 지시문 |
| `uiDescription` | `String` | `""` | UI 컨텍스트 또는 설명적 지시문 |
| `clientUiCapabilities` | `JsonObject?` | `null` | 클라이언트가 보고한 UI 역량 — 스키마 선택적 축소에 사용 |
| `allowedComponents` | `List<String>` | `emptyList()` | 렌더링 허용 컴포넌트 ID 목록 |
| `allowedMessages` | `List<String>` | `emptyList()` | 렌더링 허용 메시지 ID 목록 |
| `includeSchema` | `Boolean` | `false` | A2UI JSON 스키마를 지시문에 직접 포함할지 여부 |
| `includeExamples` | `Boolean` | `false` | 퓨샷 예시를 지시문에 포함할지 여부 |
| `validateExamples` | `Boolean` | `false` | 로드된 예시를 스키마에 대해 미리 검증할지 여부 |

반환값은 에이전트에 전달할 완전한 시스템 프롬프트 문자열이다.

## 동작 흐름

이 파일은 인터페이스 선언만 포함한다. 구체적인 구현체는 별도 클래스에서 이 인터페이스를 구현하여 `generateSystemPrompt`의 조합 로직을 제공해야 한다.
