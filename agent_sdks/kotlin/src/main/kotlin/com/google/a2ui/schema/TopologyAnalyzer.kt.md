# agent_sdks/kotlin/src/main/kotlin/com/google/a2ui/schema/TopologyAnalyzer.kt

## 개요

A2UI 컴포넌트 카탈로그 스키마를 분석하여 컴포넌트 타입별 필수 필드 집합과 참조 필드 집합을 추출하고, 컴포넌트 그래프의 위상(topology) 분석을 수행하는 내부(internal) 싱글턴이다. 버전에 따라 스키마 탐색 경로가 다르며(VERSION_0_8은 중첩된 `surfaceUpdate` 경로, 이외 버전은 `catalogSchema`의 `components` 키 직접 참조), `SchemaInspector`에게 실제 그래프 순회 로직을 위임한다. 테스트 외부에서는 `Validator.kt`의 `A2uiTopologyValidator`를 통해서만 호출된다.

## 의존성

### 외부 패키지
- `java.util.logging.Logger` — 오류 로그 출력
- `kotlinx.serialization.json.JsonArray` — JSON 배열 파싱
- `kotlinx.serialization.json.JsonElement` — JSON 원소 기반 타입
- `kotlinx.serialization.json.JsonObject` — JSON 오브젝트 파싱
- `kotlinx.serialization.json.JsonPrimitive` — JSON 원시값 판별

### 저장소 내부 모듈
- `A2uiCatalog` — [`Catalog.kt`](Catalog.kt.md) (동일 패키지, `A2uiCatalog` 데이터 클래스 및 `A2uiVersion` 열거형 정의)
- `A2uiConstants` — [`A2uiConstants.kt`](A2uiConstants.kt.md) (`CATALOG_COMPONENTS_KEY` 등 상수)
- `SchemaInspector` — [`SchemaInspector.kt`](SchemaInspector.kt.md) (그래프 순회, 참조 필드 추출, 인접 리스트 갱신)
- 패키지 내부 상수 (`PROP_COMPONENT`, `PROP_PROPERTIES`, `PROP_ITEMS`, `COMBINATOR_ALL_OF`, `COMBINATOR_ONE_OF`, `COMBINATOR_ANY_OF`) — `SchemaInspector.kt`에 `internal const val`로 정의됨

## Exports

`internal object TopologyAnalyzer` 이므로 같은 모듈 내에서만 접근 가능하다. 모듈 외부로 내보내는 공개 API는 없다.

| 이름 | 종류 | 접근성 |
|------|------|--------|
| `TopologyAnalyzer` | object | internal |
| `TopologyAnalyzer.extractComponentRequiredFields` | 함수 | (internal) public |
| `TopologyAnalyzer.extractComponentRefFields` | 함수 | (internal) public |
| `TopologyAnalyzer.analyzeTopology` | 함수 | (internal) public |

## 상세 명세

### object TopologyAnalyzer

`internal` 가시성. `package com.google.a2ui.schema`.

#### 필드
- `logger: Logger` (private) — `TopologyAnalyzer::class.java.name`으로 초기화된 JUL 로거.

#### fun extractComponentRequiredFields(catalog: A2uiCatalog): Map<String, Set<String>>

카탈로그의 컴포넌트 스키마로부터 각 컴포넌트 타입명을 키로, 그 타입에서 JSON Schema `required` 배열에 선언된 필드 이름 집합을 값으로 하는 맵을 반환한다. `component` 필드명 자체는 결과에서 제외한다.

동작 단계:
1. `extractComponents(catalog)`를 호출하여 컴포넌트 맵(`JsonObject`)을 얻는다. null이면 빈 맵을 반환한다.
2. 각 `(compName, compSchemaElem)` 쌍에 대해 `requiredFields: MutableSet<String>`을 초기화하고 내부 재귀 함수 `extractFromProps(cs: JsonElement)`를 호출한다.
3. `extractFromProps`는 `cs`가 `JsonObject`가 아니면 즉시 반환. `cs["required"]`를 `JsonArray`로 캐스팅한 후, 각 요소가 `JsonPrimitive`이고 `isString`이며 값이 `"component"`가 아닌 경우 `requiredFields`에 추가한다.
4. 이후 `allOf`, `oneOf`, `anyOf` 각각에 대해 해당 `JsonArray`의 각 원소로 재귀적으로 `extractFromProps`를 호출한다.
5. `requiredFields`가 비어 있지 않은 경우에만 `reqMap[compName] = requiredFields`로 저장한다.
6. 최종 `reqMap`을 반환한다.

#### private fun extractComponents(catalog: A2uiCatalog): JsonObject?

카탈로그 버전에 따라 서로 다른 경로로 컴포넌트 목록 `JsonObject`를 추출한다.

동작 단계:
1. `catalog.version == A2uiVersion.VERSION_0_8`이면 `catalog.serverToClientSchema` 기준으로 다음 중첩 경로를 탐색한다:
   - `properties` → `surfaceUpdate` → `properties` → `components` → `items` → `properties` → `component` → `properties`
   - 각 단계에서 해당 키 존재 확인 후 `JsonObject`로 캐스팅. 성공하면 해당 `JsonObject`를 반환한다.
   - 탐색 중 임의 예외 발생 시 `logger.severe { ... }`로 기록한다.
2. VERSION_0_8 경로 실패 또는 다른 버전이면 `catalog.catalogSchema[A2uiConstants.CATALOG_COMPONENTS_KEY]`를 `JsonObject?`로 캐스팅하여 반환한다.

#### fun extractComponentRefFields(catalog: A2uiCatalog): Map<String, Pair<Set<String>, Set<String>>>

컴포넌트 타입별 단일 참조 필드 집합과 리스트 참조 필드 집합의 쌍을 반환한다.

동작 단계:
1. `extractComponents(catalog)`가 null이면 `emptyMap()`을 반환한다.
2. 결과 `JsonObject`를 `SchemaInspector.extractReferenceFields(allComponents)`에 전달하여 그 결과를 그대로 반환한다.

#### fun analyzeTopology(rootId: String, components: List<JsonObject>, refFieldsMap: Map<String, Pair<Set<String>, Set<String>>>, raiseOnOrphans: Boolean = false): Set<String>

컴포넌트 목록으로 인접 리스트를 구성하고 루트 노드에서 DFS 순회하여 방문된 컴포넌트 ID 집합을 반환한다.

매개변수:
- `rootId: String` — 탐색 시작점인 루트 컴포넌트 ID.
- `components: List<JsonObject>` — 검사할 컴포넌트 JSON 오브젝트 목록.
- `refFieldsMap: Map<String, Pair<Set<String>, Set<String>>>` — 컴포넌트 타입별 참조 필드 정보.
- `raiseOnOrphans: Boolean = false` — `true`이면 루트로부터 도달 불가능한 컴포넌트(고아) 존재 시 `IllegalArgumentException`을 던진다.

동작 단계:
1. 빈 `adjList`와 `allIds`를 초기화한다.
2. 각 `comp`에 대해 `SchemaInspector.updateAdjacencyList(allIds, adjList, refFieldsMap, comp)`를 호출하여 인접 리스트와 전체 ID 집합을 채운다.
3. `rootId in allIds`이면 `SchemaInspector.visit(rootId, adjList)`로 방문 집합을 구한다. 아니면 `emptySet()`으로 설정한다.
4. `raiseOnOrphans`가 `true`이면 `allIds - visited`로 고아를 계산하고, 비어 있지 않으면 정렬 후 첫 번째 고아 ID를 포함한 `IllegalArgumentException`을 던진다.
5. `visited` 집합을 반환한다.

## 동작 흐름

외부(주로 `Validator.kt`의 `A2uiTopologyValidator`)에서 `extractComponentRefFields`로 스키마 메타데이터를 한 번 읽은 뒤, `analyzeTopology`를 통해 실제 컴포넌트 인스턴스 목록의 그래프를 분석한다. 버전 분기는 `extractComponents` 내부에서 처리되어 호출자에게 투명하다. 그래프 순회 구현 자체는 `SchemaInspector`에 위임된다.
