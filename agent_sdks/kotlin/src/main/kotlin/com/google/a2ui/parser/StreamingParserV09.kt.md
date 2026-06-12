# agent_sdks/kotlin/src/main/kotlin/com/google/a2ui/parser/StreamingParserV09.kt

## 개요

A2UI v0.9 프로토콜 사양을 위한 스트리밍 파서 구현체다. 부모 추상 클래스인 `StreamingParser`를 상속하며, `createSurface` / `updateComponents` / `updateDataModel` / `deleteSurface` 네 가지 메시지 타입을 처리한다. 파싱 도중 도착하는 부분적 JSON 스트림으로부터 완성된 프로토콜 메시지를 순서대로 방출하고, 컴포넌트 그래프의 도달 가능성(reachability) 검사를 통해 고아(orphan) 컴포넌트를 필터링한다.

## 의존성

### 외부 패키지
- `kotlinx.serialization.json.*` — `JsonArray`, `JsonElement`, `JsonObject`, `JsonPrimitive`, `jsonPrimitive`

### 저장소 내부 모듈
- [`../schema/Catalog.kt`](../schema/Catalog.kt.md) — `A2uiCatalog` (카탈로그 스키마 보유 및 유효성 검사 제공)
- [`../schema/A2uiSchemaManager.kt`](../schema/A2uiSchemaManager.kt.md) — `A2uiVersion` (버전 열거형)
- `StreamingParser` (동일 패키지 `com.google.a2ui.parser`) — 부모 추상 클래스

## Exports

| 이름 | 종류 |
|---|---|
| `StreamingParserV09` | 클래스 (public) |

## 상세 명세

### 클래스 `StreamingParserV09`

```
class StreamingParserV09(
  catalog: A2uiCatalog? = null,
  schemaMappings: Map<String, String> = emptyMap(),
) : StreamingParser(catalog, schemaMappings)
```

#### 필드 및 상태

| 이름 | 타입 | 설명 |
|---|---|---|
| `version` | `A2uiVersion` (override) | `catalog?.version ?: A2uiVersion.VERSION_0_9` |
| `defaultRootId` | `String` (상속 필드, init 블록에서 설정) | `"root"` |
| `yieldedCreateSurfaces` | `MutableSet<String>` (private) | 이미 방출한 `createSurface` surfaceId 집합 |
| `placeholderComponent` | `JsonObject` (override computed) | `{"component": "Row", "children": []}` — 루트가 아직 없을 때 사용하는 빈 컴포넌트 |
| `yieldedSurfacesSet` | `MutableSet<String>` (override computed) | `yieldedCreateSurfaces`를 반환 |
| `dataModelMsgType` | `String` (override computed) | 고정값 `"updateDataModel"` |

#### init 블록

생성 즉시 `defaultRootId = "root"`를 설정한다.

---

### 메서드 `isProtocolMsg(obj: JsonObject): Boolean`

`obj`에 `"createSurface"`, `"updateComponents"`, `"updateDataModel"` 중 하나라도 키로 존재하면 `true`를 반환한다. 부모 클래스에서 스트림 조각이 프로토콜 메시지인지 판별할 때 호출된다.

---

### 메서드 `sniffMetadata()`

버퍼(`jsonBuffer`) 문자열을 검사하여 아직 완전히 파싱되지 않은 JSON에서 메타데이터를 미리 추출한다.

1. `getLatestValue("surfaceId")`가 값을 반환하면 `surfaceId`를 갱신한다.
2. `getLatestValue("root")`가 값을 반환하면 `rootId`를 갱신한다.
3. `jsonBuffer`에 각 메시지 타입 키 리터럴(`"createSurface":` 등)이 포함되면 해당 타입을 `addMsgType()`으로 등록한다.

---

### 메서드 `handleCompleteObject(obj: JsonObject, sid: String?, messages: MutableList<ResponsePart>): Boolean`

완전히 파싱된 JSON 객체 하나를 받아 메시지 타입에 따라 분기 처리한다. 항상 `true`를 반환한다.

**처리 단계**

1. **유효성 검사**: `validator?.validate(obj, strictIntegrity = false)` 호출.
2. **surfaceId 확정**: `obj["surfaceId"]` → 없으면 `updateComponents` 또는 `createSurface` 하위 객체의 `surfaceId` 순서로 탐색 → `surfaceId` 필드 갱신.
3. **`deleteSurface` (조기 버퍼링)**: `deleteSurface` 키가 있고, 해당 surface가 아직 방출되지 않았으며(`yieldedSurfacesSet`에 없음), `bufferedStartMessage`도 없으면 `pendingMessages`에 추가하고 즉시 반환.
4. **`createSurface` 처리**:
   - `root` 값으로 `rootId` 갱신 (없으면 기존 값 또는 `"root"` 사용).
   - `bufferedStartMessage = obj` 저장.
   - 해당 surfaceId가 아직 방출되지 않은 경우(`yieldedStartMessages`에 없음): `yieldMessages`로 즉시 방출, `yieldedStartMessages`·`yieldedSurfacesSet`에 추가, `bufferedStartMessage = null` 초기화.
   - `pendingMessages`에서 해당 surfaceId 항목 제거.
   - `yieldReachable(messages)` 호출로 도달 가능한 컴포넌트들 방출.
5. **`updateComponents` 처리**:
   - `addMsgType("updateComponents")` 등록.
   - `root` 값으로 `rootId` 갱신.
   - `components` 배열의 각 요소를 `seenComponents[id]`에 저장.
   - `yieldReachable(messages, checkRoot = true, raiseOnOrphans = false)` 호출.
6. **`deleteSurface` (이미 방출된 경우)**: `addMsgType("deleteSurface")` 후 `yieldMessages`로 방출.
7. **`updateDataModel` 처리**: `addMsgType("updateDataModel")` 후, 하위 `JsonObject`로 `updateDataModel(udmObj, messages)` 호출, 그 뒤 `yieldMessages`로 방출.
8. **나머지**: `yieldMessages`로 방출.

---

### 메서드 `constructSniffedDataModelMessage(activeMsgType: String, deltaMsgPayload: JsonObject): JsonObject`

`{"version": "v0.9", "<activeMsgType>": <deltaMsgPayload>}` 형태의 `JsonObject`를 반환한다. 스니핑 단계에서 부분 데이터 모델 메시지를 구성할 때 사용된다.

---

### 메서드 `constructPartialMessage(processedComponents: List<JsonObject>, activeMsgType: String): JsonObject`

`updateComponents` 형태의 부분 메시지를 구성한다.

- payload 맵에 `"components"` 키로 `JsonArray(processedComponents)` 저장.
- `surfaceId`가 있으면 payload에 추가.
- `{"version": "v0.9", "updateComponents": <payload>}` 형태로 반환.

---

### 메서드 `getActiveMsgTypeForComponents(): String?`

현재 활성 메시지 타입을 결정한다.

1. `activeMsgType`이 이미 설정되어 있으면 그 값을 반환.
2. `_msgTypes` 집합을 순회하여 `"updateComponents"` 또는 `"createSurface"`가 있으면 `activeMsgType`에 저장 후 반환.
3. 해당 타입이 없으면 `_msgTypes.firstOrNull()` 반환.

---

### 메서드 `deduplicateDataModel(m: JsonObject, strictIntegrity: Boolean): Boolean`

`updateDataModel` 메시지 내 데이터 모델 값의 중복 여부를 확인한다.

1. `m`에 `"updateDataModel"` 키가 없으면 `true` 반환.
2. `value` 하위 `JsonObject`를 추출. 없으면 `true` 반환.
3. `value`의 모든 `(k, v)` 쌍을 `yieldedDataModel[k]`와 비교: 하나라도 다르면 `isNew = true`.
4. `!isNew && strictIntegrity`이면 `false` 반환 (중복으로 간주해 방출 거부).
5. `yieldedDataModel`에 새 값들을 저장 후 `true` 반환.

## 동작 흐름

스트리밍 JSON 청크가 `StreamingParser`(부모)의 파싱 루프를 통해 처리되고, 완전한 JSON 객체가 형성될 때마다 `handleCompleteObject`가 호출된다. 이 메서드는 메시지 타입에 따라 상태(`surfaceId`, `rootId`, `seenComponents`, `bufferedStartMessage`, `pendingMessages`)를 갱신하고, `yieldReachable` 또는 `yieldMessages`를 통해 소비자에게 완성된 메시지를 전달한다. `createSurface`가 아직 도착하지 않은 상태에서 `deleteSurface`가 먼저 오면 `pendingMessages`에 보류되며, 이후 `createSurface` 처리 시 정리된다. `sniffMetadata`는 버퍼 문자열 검색으로 완전히 파싱되기 전에 메시지 타입과 surfaceId/rootId를 조기에 파악한다.
