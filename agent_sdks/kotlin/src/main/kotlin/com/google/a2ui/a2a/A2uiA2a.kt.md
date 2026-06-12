# agent_sdks/kotlin/src/main/kotlin/com/google/a2ui/a2a/A2uiA2a.kt

## 개요

A2A 프로토콜과 A2UI JSON 형식을 연결하는 정적 유틸리티 객체(singleton)다. A2UI 데이터를 담는 A2A `Part` 생성, A2UI Part 식별, 데이터 추출, `AgentExtension` 설정 생성, 그리고 클라이언트-에이전트 간 확장(extension) URI 협상 로직을 제공한다.

## 의존성

### 외부 패키지
- `io.a2a.spec.AgentExtension`
- `io.a2a.spec.DataPart`
- `io.a2a.spec.Part`
- `kotlinx.serialization.json.JsonElement`

### 저장소 내부 모듈
- `com.google.a2ui.schema.A2uiConstants` (schema 패키지, 별도 파일)

## Exports

- `A2uiA2a` (object)

## 상세 명세

### `object A2uiA2a`

#### 상수

| 이름 | 값 |
|---|---|
| `A2UI_EXTENSION_BASE_URI` | `"https://a2ui.org/a2a-extension/a2ui/v"` |
| `MIME_TYPE_KEY` | `"mimeType"` |
| `A2UI_MIME_TYPE` | `"application/a2ui+json"` |
| `DEPRECATED_A2UI_MIME_TYPE` | `"application/json+a2ui"` |

---

#### `@JvmOverloads fun createA2uiPart(a2uiData: JsonElement, version: String? = null): Part<*>`

A2UI 데이터를 담는 `DataPart`를 생성한다. `version`이 null이거나 `"0.8"`, `"0.9"`, `"v0.8"`, `"v0.9"` 중 하나이면 `DEPRECATED_A2UI_MIME_TYPE`을 사용하고, 그 외 버전이면 `A2UI_MIME_TYPE`을 사용한다. metadata 맵은 `{ MIME_TYPE_KEY: <선택된 MIME 타입> }` 형태다.

---

#### `fun isA2uiPart(part: Part<*>): Boolean`

`part`가 `DataPart` 인스턴스이고, metadata의 `MIME_TYPE_KEY` 값이 `A2UI_MIME_TYPE` 또는 `DEPRECATED_A2UI_MIME_TYPE` 중 하나이면 `true`를 반환한다.

---

#### `fun getA2uiData(part: Part<*>): JsonElement?`

`isA2uiPart(part)`가 true일 때 `(part as DataPart).data`를 `JsonElement`로 캐스팅하여 반환한다. A2UI Part가 아니면 null을 반환한다.

---

#### `fun getA2uiAgentExtension(version: String, acceptsInlineCatalogs: Boolean = false, supportedCatalogIds: List<String> = emptyList()): AgentExtension`

에이전트가 지원하는 A2UI 확장을 기술하는 `AgentExtension` 객체를 생성한다.

params 맵 조립 규칙:
- `acceptsInlineCatalogs`가 true이면 `A2uiConstants.ACCEPTS_INLINE_CATALOGS_KEY` → `true` 추가
- `supportedCatalogIds`가 비어있지 않으면 `A2uiConstants.SUPPORTED_CATALOG_IDS_KEY` → 해당 목록 추가
- params 맵이 비어있으면 null을 `AgentExtension`에 전달

`AgentExtension` 생성자 인자: 설명 문자열 `"Provides agent driven UI using the A2UI JSON format."`, params(또는 null), `isSupportRequired = false`, URI `"$A2UI_EXTENSION_BASE_URI$version"`.

---

#### `fun selectNewestA2uiExtension(requestedExtensions: List<String>, advertisedExtensions: List<String>): String?`

클라이언트가 요청한 extension URI와 에이전트가 광고하는 URI의 교집합 중 `A2UI_EXTENSION_BASE_URI`로 시작하는 항목을 구하고, 버전 숫자가 가장 높은 URI를 반환한다. 교집합이 없거나 A2UI URI가 없으면 null을 반환한다. 버전 비교는 `compareVersions`를 사용한다.

---

#### `private fun compareVersions(v1: String, v2: String): Int`

두 버전 문자열을 점(`.`)으로 분리하여 각 세그먼트를 정수로 변환한 뒤 앞에서부터 순차 비교한다. 짧은 버전은 부족한 자리를 0으로 채워 비교한다. 표준 Comparator 규약(음수/0/양수)을 따른다.

---

#### `fun tryActivateA2uiExtension(requestedExtensions: List<String>, advertisedExtensions: List<String>, addActivatedExtension: (String) -> Unit): String?`

`selectNewestA2uiExtension`으로 최적 URI를 선택한 뒤, 선택된 URI가 있으면 `addActivatedExtension(uri)` 콜백을 호출하고 `A2UI_EXTENSION_BASE_URI` 접두사를 제거한 버전 문자열을 반환한다. 선택된 URI가 없으면 null을 반환한다.

## 동작 흐름

에이전트 초기화 시 `getA2uiAgentExtension`으로 광고할 확장을 생성하고, 요청 수신 시 `tryActivateA2uiExtension`으로 클라이언트와 협상한다. 응답 생성 시 `createA2uiPart`로 A2UI 데이터를 A2A Part로 포장하고, 수신 측에서는 `isA2uiPart`와 `getA2uiData`로 이를 식별·추출한다.
