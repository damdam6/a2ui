# agent_sdks/kotlin/src/main/kotlin/com/google/a2ui/parser/StreamingParserV08.kt

## 개요

A2UI v0.8 프로토콜 명세를 구현하는 구체적인 스트리밍 파서 클래스다. `StreamingParser` 추상 클래스를 상속하며, v0.8 특유의 메시지 타입(`beginRendering`, `surfaceUpdate`, `dataModelUpdate`, `deleteSurface`)의 처리 로직과 플레이스홀더 컴포넌트 구조를 정의한다. 서피스 생명주기(시작 → 컴포넌트 업데이트 → 데이터 모델 갱신 → 삭제)를 관리하며, `beginRendering` 메시지 수신 전 도착한 메시지는 대기열(`pendingMessages`)에 보관했다가 순서에 맞게 처리한다.

## 의존성

### 외부 패키지
- `kotlinx.serialization.json` — `JsonArray`, `JsonElement`, `JsonObject`, `JsonPrimitive`, `jsonPrimitive`

### 저장소 내부 모듈
- [`StreamingParser.kt`](StreamingParser.kt.md) — 부모 추상 클래스 (`StreamingParser`)
- [`../schema/Catalog.kt`](../schema/Catalog.kt.md) — `A2uiCatalog`, `A2uiVersion`

## Exports

- `StreamingParserV08` (클래스) — 이 파일의 유일한 공개 선언

---

## 상세 명세

### 클래스 `StreamingParserV08`

**시그니처:**
```
class StreamingParserV08(
  catalog: A2uiCatalog? = null,
  schemaMappings: Map<String, String> = emptyMap(),
) : StreamingParser(catalog, schemaMappings)
```

#### 필드 및 속성

| 이름 | 타입 | 값 / 설명 |
|---|---|---|
| `version` | `A2uiVersion` (override) | `catalog?.version ?: A2uiVersion.VERSION_0_8` |
| `yieldedBeginRenderingSurfaces` | `MutableSet<String>` (private) | `beginRendering` 메시지를 이미 방출한 서피스 ID 집합 |
| `yieldedSurfacesSet` | `MutableSet<String>` (override) | `yieldedBeginRenderingSurfaces`의 getter 위임 |
| `dataModelMsgType` | `String` (override) | 고정값 `"dataModelUpdate"` |

---

#### 속성 `placeholderComponent: JsonObject` (override)

아직 수신되지 않은 자식 컴포넌트 자리에 삽입되는 로딩 플레이스홀더.

구조:
```
{
  "component": {
    "Row": {
      "children": {
        "explicitList": []
      }
    }
  }
}
```
빈 `explicitList`를 가진 `Row` 컴포넌트로, 자식이 없는 빈 행을 나타낸다. 매 호출마다 새 `JsonObject` 인스턴스를 생성한다.

---

#### 함수 `isProtocolMsg(obj: JsonObject): Boolean` (override)

`obj`가 `"beginRendering"`, `"surfaceUpdate"`, `"dataModelUpdate"`, `"deleteSurface"` 중 하나 이상의 키를 포함하면 `true` 반환.

---

#### 함수 `sniffMetadata()` (override)

`jsonBuffer`를 스캔해 다음 메타데이터를 추출한다.
- `getLatestValue("surfaceId")`가 non-null이면 `surfaceId` 설정.
- `getLatestValue("root")`가 non-null이면 `rootId` 설정.
- `jsonBuffer`에 `"\"beginRendering\":"` 부분 문자열이 있으면 `addMsgType("beginRendering")`.
- `"\"surfaceUpdate\":"` → `addMsgType("surfaceUpdate")`.
- `"\"dataModelUpdate\":"` → `addMsgType("dataModelUpdate")`.
- `"\"deleteSurface\":"` → `addMsgType("deleteSurface")`.

---

#### 함수 `handleCompleteObject(obj, sid, messages): Boolean` (override)

완성된 JSON 객체를 메시지 타입에 따라 처리한다. 항상 `true`를 반환한다.

**사전 처리 — surfaceId 결정:**

`obj["surfaceId"]` 또는 `surfaceId` 필드로 `currentSid` 초기화. 그 후:
- `"surfaceUpdate"` 키가 있으면 `obj["surfaceUpdate"]["surfaceId"]`로 덮어쓰기 시도.
- `"beginRendering"` 키가 있으면 `obj["beginRendering"]["surfaceId"]`로 덮어쓰기 시도.
- `"deleteSurface"` 키가 있으면: 값이 `JsonPrimitive`(문자열)이면 해당 값; `JsonObject`이면 `["surfaceId"]` 값 사용.

`surfaceId = currentSid` 설정 후 `effectiveSid = surfaceId ?: "unknown"`.

**`deleteSurface` 처리 (선검사):**

`yieldedSurfacesSet`에 `effectiveSid`가 있거나 `bufferedStartMessage != null`이면 `deleteSurface(effectiveSid)` 호출.

**삭제된 서피스 단락(short-circuit):**

`deletedSurfaces`에 `effectiveSid`가 있으면 즉시 `true` 반환.

**대기열 처리 (surfaceUpdate / deleteSurface, 아직 미시작 서피스):**

`"surfaceUpdate"` 또는 `"deleteSurface"`가 `obj`에 있고, `yieldedSurfacesSet`에 `effectiveSid`가 없으며 `bufferedStartMessage`도 null이면: `pendingMessages[effectiveSid]`에 `obj`를 추가하고 `true` 반환.

**`beginRendering` 처리:**

1. `brVal = obj["beginRendering"]`에서 `surfaceId`와 `root` 추출 및 설정 (`rootId`가 null이면 `"root"` 기본값 사용).
2. `bufferedStartMessage = obj` 저장.
3. 해당 서피스의 시작 메시지를 아직 방출하지 않았으면(`yieldedStartMessages` 미포함): `yieldMessages(listOf(obj), messages)`, `yieldedStartMessages.add(effectiveSid)`, `yieldedSurfacesSet.add(effectiveSid)`, `bufferedStartMessage = null`.
4. `pendingMessages.remove(effectiveSid)`로 대기 중인 메시지들을 꺼내 `handleCompleteObject`로 재귀 처리.
5. `yieldReachable(messages)` 호출.
6. `true` 반환.

**`surfaceUpdate` 처리:**

1. `addMsgType("surfaceUpdate")` 호출.
2. `obj["surfaceUpdate"]["components"]` (JsonArray)에서 각 컴포넌트 객체를 `seenComponents[id] = compElem`으로 등록.
3. `yieldReachable(messages, checkRoot = true, raiseOnOrphans = false)` 호출.
4. `true` 반환.

**`dataModelUpdate` 처리:**

1. `addMsgType("dataModelUpdate")` 호출.
2. `obj["dataModelUpdate"]`를 `updateDataModel`로 처리.
3. `yieldMessages(listOf(obj), messages)` 호출.
4. `yieldReachable(messages, checkRoot = false, raiseOnOrphans = false)` 호출.
5. `true` 반환.

**`deleteSurface` 처리 (최종):**

`yieldMessages(listOf(obj), messages)` 호출 후 `true` 반환.

**그 외 메시지:**

`yieldMessages(listOf(obj), messages)` 호출 후 `true` 반환.

---

#### 함수 `constructPartialMessage(processedComponents, activeMsgType): JsonObject` (override)

부분 컴포넌트 목록으로 `surfaceUpdate` 메시지를 구성한다.

```
{
  "surfaceUpdate": {
    "surfaceId": <surfaceId>,  // surfaceId가 null이 아닐 때만 포함
    "components": [<processedComponents...>]
  }
}
```

`activeMsgType` 파라미터는 현재 구현에서 사용되지 않으며 항상 `"surfaceUpdate"` 래퍼를 생성한다.

---

#### 함수 `getActiveMsgTypeForComponents(): String?` (override)

활성 메시지 타입 결정 로직:
1. `activeMsgType`이 null이 아니면 반환.
2. `_msgTypes`를 순회하며 `"surfaceUpdate"` 또는 `"beginRendering"`이 있으면 해당 값을 `activeMsgType`으로 설정하고 반환.
3. 위 조건에 해당하는 타입이 없으면 `_msgTypes.firstOrNull()` 반환.

---

#### 함수 `deduplicateDataModel(m: JsonObject, strictIntegrity: Boolean): Boolean` (override)

`"dataModelUpdate"` 키가 없으면 `true` 반환(처리 허용). 있는 경우:

1. `m["dataModelUpdate"]`에서 `"contents"` 필드 추출.
2. `contents`가 `JsonArray`이면: 각 항목에서 `"key"` 와 `"valueString"`, `"valueNumber"`, `"valueBoolean"`, `"valueMap"` 중 하나를 추출해 `contentsDict` 구성.
3. `contents`가 `JsonObject`이면: 직접 `contentsDict`에 복사.
4. `contentsDict`가 비어있지 않으면: `yieldedDataModel`과 비교해 하나라도 다른 값이 있으면 `isNew = true`.
5. `!isNew && strictIntegrity == true`이면 `false` 반환 (중복 → 방출 생략).
6. `yieldedDataModel.putAll(contentsDict)` 갱신.
7. `true` 반환.

---

## 동작 흐름

1. **인스턴스 생성:** `StreamingParser.create(catalog)`를 통해 `catalog.version`이 `VERSION_0_9`가 아닐 때 생성된다.
2. **메타데이터 스니핑:** 스트리밍 중 `sniffMetadata`가 호출될 때마다 `jsonBuffer`에서 `surfaceId`, `rootId`, 메시지 타입 키를 탐지한다.
3. **beginRendering 수신:** 서피스 시작을 알리는 메시지. 이 메시지 수신 전에 도착한 `surfaceUpdate`/`deleteSurface` 는 `pendingMessages`에 대기 후, `beginRendering` 처리 시점에 순서대로 재처리된다.
4. **surfaceUpdate 수신:** 컴포넌트 목록을 `seenComponents`에 등록한 후 `yieldReachable`을 `checkRoot = true`로 호출해 root부터 도달 가능한 컴포넌트만 `surfaceUpdate` 래퍼 메시지로 방출한다.
5. **부분 스트리밍 방출:** JSON 블록이 닫히지 않은 상태에서도 `sniffPartialComponent`와 `yieldReachable`을 통해 이미 수신된 컴포넌트가 있으면 부분 `surfaceUpdate` 메시지를 먼저 방출한다.
6. **dataModelUpdate 수신:** 중복 검사(`deduplicateDataModel`) 후 방출; `sniffPartialDataModel`을 통한 부분 방출도 지원.
7. **deleteSurface 수신:** 서피스 관련 누적 데이터를 정리하고 삭제 메시지를 방출한다.
