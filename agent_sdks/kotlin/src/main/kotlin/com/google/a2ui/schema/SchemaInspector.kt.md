# agent_sdks/kotlin/src/main/kotlin/com/google/a2ui/schema/SchemaInspector.kt

## 개요

A2UI 카탈로그 스키마 구조를 검사·분석하는 내부 유틸리티다. JSON 스키마에서 컴포넌트 간 참조 관계(단일 자식 참조, 자식 목록 참조)를 휴리스틱으로 탐지하고, 이를 기반으로 컴포넌트 그래프의 인접 리스트를 구성한다. 또한 DFS 기반의 순환 참조 감지 및 최대 깊이 제한을 포함한 방문 함수를 제공한다. 파일 하단에는 이전 최상위 함수에서 `SchemaInspector` 객체로 위임하는 deprecated 래퍼들이 선언되어 있다.

## 의존성

### 외부 패키지
- `kotlinx.serialization.json.*` — `JsonArray`, `JsonElement`, `JsonObject`, `JsonPrimitive`, `jsonPrimitive`

### 저장소 내부 모듈
없음.

## Exports

| 이름 | 종류 | 가시성 |
|---|---|---|
| `KEY_DOLLAR_REF` | `const val` | internal |
| `PROP_PROPERTIES` | `const val` | internal |
| `PROP_TYPE` | `const val` | internal |
| `PROP_TITLE` | `const val` | internal |
| `PROP_ITEMS` | `const val` | internal |
| `COMBINATOR_ONE_OF` | `const val` | internal |
| `COMBINATOR_ANY_OF` | `const val` | internal |
| `COMBINATOR_ALL_OF` | `const val` | internal |
| `TYPE_STRING` | `const val` | internal |
| `TYPE_OBJECT` | `const val` | internal |
| `TYPE_ARRAY` | `const val` | internal |
| `MAX_GLOBAL_DEPTH` | `const val` | internal (값: `50`) |
| `PROP_COMPONENT` | `const val` | internal |
| `PROP_CHILD` | `const val` | internal |
| `PROP_CHILDREN` | `const val` | internal |
| `PROP_CONTENT_CHILD` | `const val` | internal |
| `PROP_ENTRY_POINT_CHILD` | `const val` | internal |
| `PROP_COMPONENT_ID` | `const val` | internal |
| `PROP_EXPLICIT_LIST` | `const val` | internal |
| `PROP_TEMPLATE` | `const val` | internal |
| `TITLE_COMPONENT_ID` | `const val` | internal (값: `"ComponentId"`) |
| `TITLE_CHILD_LIST` | `const val` | internal (값: `"ChildList"`) |
| `SchemaInspector` | object | internal |
| `isComponentIdRef` | 함수 (deprecated) | internal |
| `isChildListRef` | 함수 (deprecated) | internal |
| `extractReferenceFields` | 함수 (deprecated) | internal |
| `getComponentReferences` | 함수 (deprecated) | internal |
| `updateAdjacencyList` | 함수 (deprecated) | internal |
| `visit` | 함수 (deprecated) | internal |

## 상세 명세

### 파일 레벨 상수

모두 `internal const val`로 선언된 문자열/정수 상수다. JSON 스키마 키 이름과 컴포넌트 프로퍼티 이름, 타입 리터럴, 최대 재귀 깊이를 담는다.

| 상수 | 값 |
|---|---|
| `KEY_DOLLAR_REF` | `"$ref"` |
| `PROP_PROPERTIES` | `"properties"` |
| `PROP_TYPE` | `"type"` |
| `PROP_TITLE` | `"title"` |
| `PROP_ITEMS` | `"items"` |
| `COMBINATOR_ONE_OF` | `"oneOf"` |
| `COMBINATOR_ANY_OF` | `"anyOf"` |
| `COMBINATOR_ALL_OF` | `"allOf"` |
| `TYPE_STRING` | `"string"` |
| `TYPE_OBJECT` | `"object"` |
| `TYPE_ARRAY` | `"array"` |
| `MAX_GLOBAL_DEPTH` | `50` |
| `PROP_COMPONENT` | `"component"` |
| `PROP_CHILD` | `"child"` |
| `PROP_CHILDREN` | `"children"` |
| `PROP_CONTENT_CHILD` | `"contentChild"` |
| `PROP_ENTRY_POINT_CHILD` | `"entryPointChild"` |
| `PROP_COMPONENT_ID` | `"componentId"` |
| `PROP_EXPLICIT_LIST` | `"explicitList"` |
| `PROP_TEMPLATE` | `"template"` |
| `TITLE_COMPONENT_ID` | `"ComponentId"` |
| `TITLE_CHILD_LIST` | `"ChildList"` |

---

### internal object `SchemaInspector`

#### private val `HEURISTIC_SINGLE_REFS`

`setOf("child", "contentChild", "entryPointChild", "detail", "summary", "root")` — 단일 컴포넌트 참조로 간주할 프로퍼티 이름 집합.

#### private val `HEURISTIC_LIST_REFS`

`setOf("children", "explicitList", "template")` — 자식 목록으로 간주할 프로퍼티 이름 집합.

---

#### fun `isComponentIdRef(propSchema: JsonElement): Boolean`

프로퍼티 스키마가 단일 `ComponentId` 참조를 나타내는지 휴리스틱으로 판단한다.

1. `JsonObject`가 아니면 `false`.
2. `$ref` 값이 `"ComponentId"` 또는 `"child"` 또는 `"/child"` 포함 시 `true`.
3. `type == "string"` 이고 `title == "ComponentId"`이면 `true`.
4. `oneOf` / `anyOf` / `allOf` 배열 중 하나라도 `isComponentIdRef`를 재귀적으로 만족하는 항목이 있으면 `true`.
5. 위 조건 모두 불충족 시 `false`.

---

#### fun `isChildListRef(propSchema: JsonElement): Boolean`

프로퍼티 스키마가 자식 목록 참조를 나타내는지 휴리스틱으로 판단한다.

1. `JsonObject`가 아니면 `false`.
2. `$ref` 값이 `"ChildList"` 또는 `"children"` 또는 `"/children"` 포함 시 `true`.
3. `type == "object"`이고 `properties`에 `"explicitList"`, `"template"`, `"componentId"` 중 하나라도 있으면 `true`.
4. `type == "array"`이고 `items`가 `isComponentIdRef`를 만족하면 `true`.
5. 조합자(`oneOf` / `anyOf` / `allOf`) 중 하나라도 `isChildListRef`를 재귀 만족하면 `true`.
6. 위 조건 모두 불충족 시 `false`.

---

#### fun `extractReferenceFields(allComponents: JsonObject): MutableMap<String, Pair<Set<String>, Set<String>>>`

카탈로그의 모든 컴포넌트를 순회하여 각 컴포넌트의 단일 참조 프로퍼티 집합과 목록 참조 프로퍼티 집합을 추출한다. 반환값은 `compName → (singleRefs, listRefs)` 매핑이다.

**내부 재귀 함수 `extractFromProps(cs: JsonElement)`**:
- `cs`가 `JsonObject`가 아니면 반환.
- `"properties"` 하위 각 `(propName, propSchema)` 쌍에 대해:
  - `isComponentIdRef(propSchema) == true` 또는 propName이 `child`, `contentChild`, `entryPointChild`이면 `singleRefs`에 추가.
  - `isChildListRef(propSchema) == true` 또는 propName이 `"children"`이면 `listRefs`에 추가.
- `allOf`, `oneOf`, `anyOf` 배열의 각 항목에 대해 `extractFromProps` 재귀 호출.

`singleRefs`나 `listRefs`가 비어있지 않은 컴포넌트만 결과 맵에 포함.

---

#### fun `getComponentReferences(component: JsonObject, refFieldsMap: Map<String, Pair<Set<String>, Set<String>>>): Sequence<Pair<String, String>>`

컴포넌트 JSON 객체로부터 참조하는 자식 컴포넌트 ID들을 `Sequence`로 방출한다. 각 쌍은 `(참조된 컴포넌트 ID, 필드 경로 문자열)` 형태다.

`component["component"]` 값에 따라 분기:
- `JsonPrimitive` (문자열): `getRefsRecursively(compType, component, refFieldsMap)` 위임.
- `JsonObject`: 각 `(cType, cProps)` 쌍에서 `cProps`가 `JsonObject`이면 `getRefsRecursively` 위임.

---

#### private fun `getRefsRecursively(compType: String, props: JsonObject, refFieldsMap: ...): Sequence<Pair<String, String>>`

`refFieldsMap[compType]`에서 `(singleRefs, listRefs)` 추출. props의 각 `(key, value)` 쌍을 처리한다:

**단일 참조** (`key in singleRefs || key in HEURISTIC_SINGLE_REFS`):
- `value`가 문자열 `JsonPrimitive`: `(value.content, key)` 방출.
- `value`가 `JsonObject`이고 `componentId` 키 있으면: `(componentId값, "$key.componentId")` 방출.

**목록 참조** (`key in listRefs || key in HEURISTIC_LIST_REFS`):
- `value`가 `JsonArray`: 각 문자열 항목에 대해 `(item.content, key)` 방출.
- `value`가 `JsonObject`:
  - `explicitList` 있으면: 배열 내 각 문자열 항목 → `(id, "$key.explicitList")` 방출.
  - `template` 있으면: `template.componentId` → `(id, "$key.template.componentId")` 방출.
  - `componentId` 있으면: `(id, "$key.componentId")` 방출.

**그 외**: `value`가 `JsonArray`이면 인덱스별로 항목이 `JsonObject`이고 `"child"` 키가 있으면 `(child값, "$key[$idx].child")` 방출.

---

#### fun `updateAdjacencyList(allIds: MutableSet<String>, adjList: MutableMap<String, MutableList<String>>, refFieldsMap: ..., comp: JsonObject)`

`comp["id"]`가 없으면 즉시 반환. 있으면 `allIds`에 추가, `adjList[compId]` 이웃 목록을 생성.

`getComponentReferences`를 순회하여 각 참조 ID를 이웃 목록에 추가. 자기 자신을 참조하면 (`refId == compId`) `IllegalArgumentException` 발생.

---

#### fun `visit(nodeId: String, adjList: MutableMap<String, MutableList<String>>): Set<String>`

주어진 노드에서 DFS 탐색을 시작하여 도달 가능한 모든 노드 ID 집합을 반환한다. 내부적으로 `dfs` 호출.

---

#### private fun `dfs(nodeId, visited, adjList, recursionStack, depth)`

재귀 DFS 구현. 처리 단계:

1. `depth > MAX_GLOBAL_DEPTH (50)`: `IllegalArgumentException` 발생.
2. `visited`와 `recursionStack`에 `nodeId` 추가.
3. `adjList[nodeId]`의 각 이웃에 대해:
   - 방문하지 않았으면 재귀 호출.
   - 방문했는데 `recursionStack`에 있으면 순환 참조 → `IllegalArgumentException` 발생.
4. `recursionStack`에서 `nodeId` 제거 (backtracking).

---

### Deprecated 최상위 함수들

6개의 internal 함수(`isComponentIdRef`, `isChildListRef`, `extractReferenceFields`, `getComponentReferences`, `updateAdjacencyList`, `visit`)가 각각 `@Deprecated("Use SchemaInspector directly", ReplaceWith(...))`로 선언되어 있다. 이들은 단순히 `SchemaInspector.해당함수(...)` 를 호출한다.

## 동작 흐름

스트리밍 파서와 검증기가 카탈로그 스키마로부터 컴포넌트 참조 구조를 파악할 때 이 모듈을 사용한다. 먼저 `extractReferenceFields`로 스키마 기반의 참조 필드 맵을 구성하고, 파싱 중 도착한 컴포넌트에 대해 `updateAdjacencyList`로 그래프를 갱신하며, `visit`을 통해 루트에서 도달 가능한 노드 집합을 구해 고아 컴포넌트를 필터링한다. 자기 참조와 순환 참조는 즉시 예외로 차단되고, 50 depth 초과도 예외로 차단된다.
