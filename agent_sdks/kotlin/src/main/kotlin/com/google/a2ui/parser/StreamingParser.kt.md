# agent_sdks/kotlin/src/main/kotlin/com/google/a2ui/parser/StreamingParser.kt

## 개요

A2UI 프로토콜의 스트리밍 JSON 파싱을 담당하는 추상 기반 클래스다. LLM이 생성하는 텍스트 스트림에서 `<a2ui>...</a2ui>` 구분자를 탐지하고, 그 안의 JSON을 점진적으로 파싱하여 UI 컴포넌트 메시지를 산출(yield)한다. 버전별 구현(v0.8, v0.9)은 이 클래스를 상속하며, `create` 팩토리 메서드로 적절한 구현체를 생성한다. 불완전한 JSON 조각을 안전하게 수리(fixJson)하고, 컴포넌트 위상 분석(topology)을 통해 도달 가능한 컴포넌트만 부분 메시지로 방출하는 복잡한 스트리밍 로직을 포함한다.

## 의존성

### 외부 패키지
- `kotlinx.serialization.json` — `Json`, `JsonObject`, `JsonArray`, `JsonElement`, `JsonPrimitive`, 확장 속성(`jsonArray`, `jsonObject`, `jsonPrimitive`)
- `kotlinx.serialization.SerializationException`
- `java.util.logging.Logger`

### 저장소 내부 모듈
- [`../schema/Catalog.kt`](../schema/Catalog.kt.md) — `A2uiCatalog`, `A2uiVersion`, `A2uiConstants`
- [`../schema/Validator.kt`](../schema/Validator.kt.md) — `A2uiValidator`
- [`../schema/TopologyAnalyzer.kt`](../schema/TopologyAnalyzer.kt.md) — `TopologyAnalyzer`
- (동일 패키지) `StreamingParserV09` — `create` 팩토리 메서드에서 직접 참조
- (동일 패키지) `StreamingParserV08` — `create` 팩토리 메서드에서 직접 참조
- (동일 패키지) `ResponsePart` — `Parser.kt` 에 정의

## Exports

- `StreamingParser` (추상 클래스) — 이 파일의 유일한 공개 선언

---

## 상세 명세

### 클래스 `StreamingParser`

**시그니처:**
```
abstract class StreamingParser(
  protected val catalog: A2uiCatalog?,
  protected val schemaMappings: Map<String, String> = emptyMap(),
)
```

#### 생성자 초기화 (필드 및 상태)

생성자 본문에서 `catalog`를 사용해 다음 필드들이 즉시 초기화된다.

| 필드 | 타입 | 초기값 / 의미 |
|---|---|---|
| `version` | `A2uiVersion?` | `catalog?.version` (서브클래스에서 override 가능) |
| `refFieldsMap` | `Map<String, Pair<Set<String>, Set<String>>>` | `TopologyAnalyzer.extractComponentRefFields(catalog)` 또는 `emptyMap()` |
| `requiredFieldsMap` | `Map<String, Set<String>>` | `TopologyAnalyzer.extractComponentRequiredFields(catalog)` 또는 `emptyMap()` |
| `cuttableKeys` | `Set<String>` | `catalog?.cuttableKeys` 또는 `emptySet()` |
| `validator` | `A2uiValidator?` | `A2uiValidator(catalog, schemaMappings)` 또는 null |

**파싱 상태 필드:**

| 필드 | 타입 | 역할 |
|---|---|---|
| `foundDelimiter` | `Boolean` | 현재 `<a2ui>` 열기 태그를 찾았는지 여부 |
| `buffer` | `StringBuilder` | 구분자 탐지용 입력 버퍼 |
| `jsonBuffer` | `StringBuilder` | 파싱 중인 JSON 내용 버퍼 |
| `braceStack` | `MutableList<Pair<String, Int>>` | 중첩 구조 추적용 스택 (`"{"` or `"["`, 버퍼 내 시작 인덱스) |
| `braceCount` | `Int` | 현재 중첩 깊이 |
| `inTopLevelList` | `Boolean` | 최상위 `[...]` 배열 내부인지 여부 |
| `inString` | `Boolean` | 현재 JSON 문자열 내부인지 여부 |
| `stringEscaped` | `Boolean` | 직전 문자가 백슬래시였는지 여부 |

**추적/누적 상태 필드:**

| 필드 | 타입 | 역할 |
|---|---|---|
| `seenComponents` | `MutableMap<String, JsonObject>` | id → 컴포넌트 객체 (수신된 모든 컴포넌트) |
| `yieldedDataModel` | `MutableMap<String, JsonElement>` | 이미 방출된 데이터 모델 키 → 값 (중복 방출 방지) |
| `deletedSurfaces` | `MutableSet<String>` | 삭제된 서피스 ID 집합 |
| `yieldedIds` | `MutableMap<String, MutableSet<String>>` | surfaceId → 방출된 컴포넌트 ID 집합 |
| `yieldedContents` | `MutableMap<Pair<String, String>, JsonObject>` | (surfaceId, componentId) → 방출된 컴포넌트 객체 |
| `rootIds` | `MutableMap<String, String>` | surfaceId → root 컴포넌트 ID |
| `defaultRootId` | `String?` | 기본 root ID |
| `unboundRootId` | `String?` | surfaceId가 아직 결정되지 않은 경우의 임시 root ID |
| `pendingMessages` | `MutableMap<String, MutableList<JsonObject>>` | beginRendering 전에 도착한 메시지를 대기 |
| `bufferedStartMessage` | `JsonObject?` | beginRendering 메시지 버퍼 |
| `topologyDirty` | `Boolean` | 위상 재계산 필요 여부 |
| `foundValidJsonInBlock` | `Boolean` | 현재 A2UI 블록에서 유효한 JSON을 하나 이상 파싱했는지 여부 |
| `yieldedStartMessages` | `MutableSet<String>` | 이미 시작 메시지를 방출한 surfaceId 집합 |
| `activeMsgType` | `String?` | 현재 활성 메시지 타입 |
| `_msgTypes` | `MutableList<String>` | 현재 블록에서 감지된 메시지 타입 목록 |

---

#### 속성 `surfaceId: String?`

커스텀 getter/setter를 가진 공개 속성. 설정 시, `unboundRootId`가 존재하면 `rootIds[value] = unboundRootId`로 바인딩하고 `unboundRootId`를 null로 지운다.

#### 속성 `rootId: String?`

커스텀 getter/setter.
- **get:** `surfaceId`가 있으면 `rootIds[surfaceId] ?: defaultRootId`를 반환; 없으면 `unboundRootId ?: defaultRootId` 반환.
- **set:** `surfaceId`가 있으면 `rootIds[surfaceId] = value` (null이면 제거); 없으면 `unboundRootId = value`.

#### 속성 `msgTypes: List<String>` (읽기 전용)

`_msgTypes`의 불변 뷰.

---

#### 함수 `addMsgType(msgType: String)`

`msgType`이 `_msgTypes`에 없으면 추가한다. 그 후 `msgType`이 `"surfaceUpdate"`, `"updateComponents"`, `"createSurface"` 중 하나이면 `activeMsgType`을 해당 값으로 갱신한다.

---

#### 추상/오버라이드 대상 멤버 (서브클래스 계약)

| 멤버 | 종류 | 설명 |
|---|---|---|
| `placeholderComponent` | `abstract val JsonObject` | 아직 수신되지 않은 자식 컴포넌트 대신 삽입할 플레이스홀더 객체 |
| `yieldedSurfacesSet` | `abstract val MutableSet<String>` | 이미 시작 메시지를 방출한 서피스 집합 (구현체가 관리) |
| `dataModelMsgType` | `abstract val String` | 데이터 모델 업데이트 메시지의 키 이름 (예: `"dataModelUpdate"`) |
| `isProtocolMsg(obj)` | `abstract fun` | 객체가 프로토콜 메시지인지 여부 판별 |
| `getActiveMsgTypeForComponents()` | `abstract fun String?` | 컴포넌트용 활성 메시지 타입 반환 |
| `handleCompleteObject(obj, sid, messages)` | `abstract fun Boolean` | 완성된 JSON 객체를 처리; `true` 반환 시 기본 처리 생략 |
| `constructPartialMessage(components, activeMsgType)` | `abstract fun JsonObject` | 처리된 컴포넌트 목록으로 부분 메시지 JsonObject 구성 |
| `sniffMetadata()` | `abstract fun` | `jsonBuffer` 를 스캔해 메타데이터(surfaceId, root 등) 추출 |
| `deduplicateDataModel(m, strictIntegrity)` | `open fun Boolean` | 기본 구현은 항상 `true` 반환; 서브클래스에서 중복 필터링 |
| `constructSniffedDataModelMessage(activeMsgType, deltaMsgPayload)` | `open fun JsonObject` | 기본 구현은 `JsonObject(mapOf(activeMsgType to deltaMsgPayload))` 반환 |

---

#### 함수 `fixJson(fragment: String): String`

불완전하게 잘린 JSON 문자열을 수리하여 파싱 가능한 형태로 만든다.

1. `fragment`를 오른쪽 공백 제거.
2. 문자를 순서대로 스캔하면서 `inStr`/`escaped` 상태를 추적하고, `{`, `[`/`}`, `]` 를 스택(`stack`)에 push/pop한다.
3. **열린 문자열(`inStr == true`):** 마지막 따옴표 위치(`lastQuoteIdx`) 이전 부분의 끝이 `":` 이면:
   - KEY_MATCH_REGEX(`"([^\"]+)"\s*:\s*$`)로 키 이름 추출.
   - 키가 `cuttableKeys`에 없으면 빈 문자열 반환 (잘린 비-cuttable 값은 무효).
   - 키가 `"valueString"`이면 실제 값 시작(`fixed[lastQuoteIdx+1:]`)이 `http://`, `https://`, `data:`, `/`로 시작하는 URL이면 빈 문자열 반환.
   - 또한 직전 200자 이내의 `"key"` 값(PREV_KEY_MATCHES_REGEX)이 `"url"`, `"link"`, `"src"`, `"href"`, `"image"` 중 하나를 포함하면 빈 문자열 반환.
   - 위 조건에 해당하지 않으면 `"` 한 글자 추가(문자열 닫기).
4. 오른쪽 공백 재 제거 후 끝이 `,`이면 제거.
5. 스택에 남은 여는 괄호들을 역순으로 닫는 괄호(`}` 또는 `]`)로 채운다.
6. 수리된 문자열 반환.

---

#### 함수 `resetJsonState()`

JSON 파싱 상태를 초기화한다: `jsonBuffer`, `braceStack`, `braceCount`, `inTopLevelList`, `inString`, `stringEscaped`, `_msgTypes`, `foundValidJsonInBlock`를 모두 초기값으로 리셋.

---

#### 함수 `deleteSurface(sid: String)`

지정 서피스와 관련된 모든 누적 데이터를 정리한다: `pendingMessages`, `yieldedIds`, `yieldedContents`에서 해당 `sid` 항목을 제거하고, `yieldedSurfacesSet`, `yieldedStartMessages`에서도 제거하며, `deletedSurfaces`에 추가한다.

---

#### 함수 `yieldMessages(messagesToYield, messages, strictIntegrity)`

**시그니처:** `protected fun yieldMessages(messagesToYield: List<JsonObject>, messages: MutableList<ResponsePart>, strictIntegrity: Boolean = true)`

메시지 목록을 검증 후 `messages`에 추가하는 내부 메서드.

1. 각 메시지에 대해 `deduplicateDataModel`로 중복 여부 검사; `false`면 건너뜀.
2. `validator`가 있으면 `validate(m, strictIntegrity)` 호출.
   - `strictIntegrity = true`이면 예외를 그대로 던짐.
   - `strictIntegrity = false`이면 예외를 잡아 `"updateComponents"` 메시지인 경우 컴포넌트별 유효성 추가 확인(`requiredFieldsMap` 기반); 모든 컴포넌트가 유효하면 방출 허용, 그렇지 않으면 건너뜀.
3. `messages`의 마지막 `ResponsePart`에 `a2uiJson`이 없으면 해당 part에 추가; 있으면 기존 리스트에 append; `messages`가 비어있으면 새 `ResponsePart` 추가.

---

#### 함수 `processChunk(chunk: String): List<ResponsePart>`

스트리밍 청크 한 단위를 처리하는 공개 진입점.

1. `chunk`를 `buffer`에 추가.
2. 루프:
   - `foundDelimiter == false`: `buffer`에서 `A2uiConstants.A2UI_OPEN_TAG`를 검색. 발견 시 태그 이전 텍스트를 `ResponsePart(text=...)`로 방출하고, 태그 이후로 `buffer`를 잘라 `foundDelimiter = true`. 미발견 시 태그의 접두사일 수 있는 부분(`keepLen`)은 남기고 나머지 텍스트를 안전하게 방출 후 루프 탈출.
   - `foundDelimiter == true`: `buffer`에서 `A2uiConstants.A2UI_CLOSE_TAG`를 검색. 발견 시 태그 이전을 `processJsonChunk`로 처리; `foundValidJsonInBlock == false`이면 예외 발생; `foundDelimiter = false`로 초기화 후 `resetJsonState` 호출. 미발견 시 닫기 태그의 접두사일 수 있는 부분을 제외하고 `processJsonChunk` 호출 후 루프 탈출.
3. 루프 종료 후 `messages` 내 모든 `ResponsePart`에 대해 동일 `surfaceId`를 가진 중복 `surfaceUpdate` 메시지를 역순 스캔으로 제거(가장 최신 것만 유지).
4. 디버그 목적으로 `chunk`에 `"s.png"` 포함 시 stderr 출력.
5. `messages` 반환.

---

#### 함수 `processJsonChunk(chunk: String, messages: MutableList<ResponsePart>)`

`jsonBuffer` 크기 한도(`MAX_JSON_BUFFER_SIZE = 5 * 1024 * 1024` bytes) 초과 시 예외 발생.

문자 단위 상태 머신:
- **최상위 리스트 미진입 상태:** `[`가 나타날 때까지 모든 문자를 무시; `[`를 만나면 `inTopLevelList = true`, `braceStack`에 push, `braceCount++`.
- **문자열 내부 상태 (`inString == true`):** escape 여부에 따라 다음 문자를 처리하고 `jsonBuffer`에 추가; `"`를 만나면 `inString = false`.
- **일반 상태:**
  - `"`: `inString = true`.
  - `{`: `braceStack`에 push, `braceCount++`. `braceCount == 0`이던 시점에는 `_msgTypes.clear()`.
  - `}`: `braceStack`에서 pop, `braceCount--`. pop한 시작 인덱스부터 현재까지의 서브 버퍼가 완전한 `{...}` 이면 `Json.parseToJsonElement`로 파싱 시도:
    - 성공 시 `foundValidJsonInBlock = true`.
    - 객체에 `"id"`와 `"component"` 키가 모두 있으면 `handlePartialComponent` 호출.
    - 최상위 객체이거나 프로토콜 메시지(`isProtocolMsg`)이면 `handleCompleteObject` 호출; 반환값이 `false`이면 `yieldMessages` 직접 호출.
    - 완성된 객체에 해당하는 `jsonBuffer` 구간 삭제 및 `braceStack` 인덱스 조정.
  - `[`: push, `braceCount++`.
  - `]`: pop(스택 top이 `"["` 인 경우), `braceCount--`. `braceCount == 0`이면 `inTopLevelList = false`.
  - 그 외: `braceCount > 0`이면 `jsonBuffer`에 추가.
- `"`, `:`, `,`, `}`, `]` 문자를 처리한 후 `sniffMetadata()` 호출.

청크 처리 완료 후:
- `foundDelimiter == true` (블록 종료): `sniffMetadata`, `sniffPartialDataModel`, `sniffPartialComponent`, `yieldReachable(checkRoot=false, raiseOnOrphans=false)` 순서로 호출; `topologyDirty = false`.
- `foundDelimiter == false` (블록 진행 중): `braceCount >= 1`이고 `jsonBuffer` 비어있지 않으면 `sniffPartialComponent`, `sniffPartialDataModel` 호출; `topologyDirty == true`이면 `yieldReachable` 후 `topologyDirty = false`.

---

#### 함수 `getLatestValue(key: String): String?`

`jsonBuffer`를 역방향으로 탐색하여 `"key": "value"` 패턴의 가장 최신 값을 반환한다. `"surfaceId"` 와 `"root"` 키는 미리 컴파일된 전용 정규식(`SURFACE_ID_REGEX`, `ROOT_ID_REGEX`)을 사용하고, 그 외 키는 `LATEST_VALUE_REGEX_CACHE`에서 캐시된 정규식을 사용한다. 매치가 버퍼 내 검색 위치 직전에서 시작해야 유효하다고 판단하며, 해당하지 않으면 더 앞 위치에서 재검색을 반복한다.

---

#### 함수 `sniffPartialComponent(messages: MutableList<ResponsePart>)`

`jsonBuffer`에 `"components"` 문자열이 없으면 즉시 반환. `braceStack`을 역순으로 순회하며 `{` 타입 항목들에 대해 해당 버퍼 구간을 `fixJson`으로 수리 후 파싱을 시도한다. `"id"`와 `"component"` 키가 모두 있는 객체를 발견하면 `handlePartialComponent`를 호출하고 루프를 종료한다.

---

#### 함수 `sniffPartialDataModel(messages: MutableList<ResponsePart>)`

`jsonBuffer`에 데이터 모델 메시지 타입 키(`"$dataModelMsgType"`)가 없으면 즉시 반환. `braceStack`을 역순으로 순회하며 `fixJson` 수리 후 파싱을 시도한다. JSON 파싱 실패 시 마지막 `,` 위치부터 거슬러 올라가며 잘라서 재시도하는 폴백 로직을 포함한다.

데이터 모델 객체 발견 시:
- `contents` 필드(JsonArray 또는 JsonObject)와 `value` 필드(JsonObject)를 검사.
- 이미 방출된 `yieldedDataModel`과 비교해 변경된 항목만 `delta`로 추출.
- `delta`가 비어있지 않으면 `surfaceId`(없으면 `"default"`)를 포함한 페이로드를 구성하고 `constructSniffedDataModelMessage`로 메시지를 만들어 `yieldMessages` 호출.
- `yieldedDataModel` 갱신 후 `updateDataModel` 호출.

---

#### 함수 `pruneIncompleteDatamodelEntries(entries: JsonElement): JsonElement`

JsonArray 형태의 데이터 모델 항목들을 순회하며, 각 항목이 `"value"`, `"valueString"`, `"valueNumber"`, `"valueBoolean"` 중 하나 또는 `"valueMap"`(재귀 처리)을 포함하고 `"key"` 필드가 있는 경우에만 결과 배열에 포함시킨다. `"valueMap"`이 있으나 재귀 처리 결과가 빈 배열이 되면 `valueMap` 필드를 제거한다.

---

#### 함수 `parseContentsToDict(rawContents: JsonElement): Map<String, JsonElement>`

데이터 모델의 `contents` 필드를 `key → value` 맵으로 변환한다.
- `JsonObject`이면 그대로 반환.
- `JsonArray`이면 각 항목에서 `"key"` 문자열과 `"value"`, `"valueString"`, `"valueNumber"`, `"valueBoolean"`, `"valueMap"` 중 하나의 값을 추출해 맵으로 구성한다. `"valueMap"`은 재귀 호출로 `JsonObject`로 래핑.

---

#### 함수 `updateDataModel(update: JsonObject, messages: MutableList<ResponsePart>)`

`update`에서 `contents` 필드를 `parseContentsToDict`로 파싱하거나, 없는 경우 `"surfaceId"`, `"root"`, `"contents"` 를 제외한 나머지 키-값 쌍으로 맵을 구성한다. 현재 구현에서는 추가 처리 없이 반환(주석: `Currently no extra marking logic needed`).

---

#### 함수 `handlePartialComponent(comp: JsonObject, messages: MutableList<ResponsePart>)`

부분적으로 수신된 컴포넌트를 `seenComponents`에 등록하는 로직.

1. `"id"` 필드가 없으면 즉시 반환.
2. `hasEmptyDict` 내부 함수: `JsonObject`나 `JsonArray` 내에 빈 `{}`가 재귀적으로 존재하면 `true` 반환.
3. `"component"` 값이 `JsonPrimitive`(문자열)이고 `hasEmptyDict(comp)` 이면 반환 (불완전한 컴포넌트).
4. `"component"` 값이 `JsonObject`이고 `requiredFieldsMap`이 비어있지 않으면: 컴포넌트 타입을 키로 삼아 `props`를 추출하고, `requiredFieldsMap[compType]`의 모든 필드가 `props`에 있는지 확인; 하나라도 없으면 반환.
5. 위 검사를 모두 통과하면 `seenComponents[compId] = comp` 등록, `topologyDirty = true`.

---

#### 함수 `yieldReachable(messages, checkRoot, raiseOnOrphans)`

**시그니처:** `protected fun yieldReachable(messages: MutableList<ResponsePart>, checkRoot: Boolean = false, raiseOnOrphans: Boolean = false)`

위상 분석을 기반으로 도달 가능한 컴포넌트를 방출한다.

1. `getActiveMsgTypeForComponents()`와 `rootId`가 null이면 반환.
2. `surfaceId`가 null이거나 해당 서피스가 아직 시작 메시지를 방출하지 않았고 `bufferedStartMessage`도 없으면 반환.
3. `TopologyAnalyzer.analyzeTopology`로 `rootId`에서 도달 가능한 ID 집합 계산.
4. `checkRoot == true`이고 root 컴포넌트가 `seenComponents`에 없으면 예외 발생.
5. 도달 가능하고 실제로 수신된 컴포넌트들을 ID 정렬 순서로 처리하며 `processComponentTopology` 호출; 이미 방출된 컴포넌트는 `inlineResolved = true`로 호출.
6. 신규 컴포넌트가 있거나 내용이 변경된 컴포넌트가 있으면 방출(shouldYield = true):
   - `bufferedStartMessage`가 있고 아직 방출하지 않았으면 먼저 방출.
   - `constructPartialMessage`로 부분 메시지 구성 후 `yieldMessages` 호출.
   - `yieldedIds`, `yieldedContents` 갱신.
7. 예외 발생 시: `raiseOnOrphans`, `"Circular"`, `"Self-reference"`, `"recursion"`, `checkRoot` 조건 중 하나라도 해당하면 재던짐, 그렇지 않으면 무시.

---

#### 함수 `getPlaceholderId(childId: String): String`

`"loading_$childId"` 문자열 반환. 아직 수신되지 않은 자식 컴포넌트의 플레이스홀더 ID 생성.

---

#### 비공개 함수 `addPlaceholderComponent(placeholderId, extraComponents, addedPlaceholderIds)`

`addedPlaceholderIds.add(placeholderId)`가 `true`(신규)일 때만 `placeholderComponent`의 복사본에 `"id"`를 `placeholderId`로 설정하여 `extraComponents`에 추가.

---

#### 함수 `processComponentTopology(comp, extraComponents, inlineResolved): JsonObject`

컴포넌트 내의 자식 참조(ID 문자열)를 점검하고 아직 수신되지 않은 자식은 플레이스홀더로 교체하는 재귀 변환.

내부 `transform(elem, parentKey)` 함수:
- **JsonObject 처리:**
  1. `"path"` 필드 처리: 값이 `/`로 시작하는 문자열이고 `version != VERSION_0_9`이면, `"componentId"` 키가 없는 경우 나머지 필드를 제거하고 `"path"`만 유지.
  2. `"path"` 값이 `/`로 시작하지 않으면 `/`를 앞에 붙이고 `"componentId"` 없는 경우 나머지 필드 제거.
  3. 컴포넌트 타입(`compType`)을 결정: `"component"` 값이 `JsonObject`이면 첫 번째 키, 아니면 문자열 값.
  4. `refFieldsMap[compType]`의 동적 참조 필드와 정적 필드 집합(`"children"`, `"explicitList"`, `"child"`, `"contentChild"`, `"entryPointChild"`, `"componentId"`)을 합쳐 `fieldsToInspect` 구성.
  5. 각 필드 값이 `JsonArray`이면: 원소 중 `seenComponents`에 없는 ID는 `getPlaceholderId`로 교체하고 플레이스홀더 추가. 변환 후 유효 자식이 없고 필드가 `"children"` 또는 `"explicitList"`이면 `jsonBuffer`에서 해당 필드 이후 `[`가 있지만 `]`가 아직 없는 경우 `"loading_children_$compId"` 플레이스홀더 삽입.
  6. 필드 값이 `JsonPrimitive`(문자열)이면: `seenComponents`에 없으면 플레이스홀더로 교체.
  7. 모든 값에 재귀 `transform` 적용.
- **JsonArray:** 각 원소에 `transform` 재귀 적용.
- **그 외:** 그대로 반환.

`"component"` 값이 `JsonObject`이면 해당 부분에만 `transform` 적용; 그렇지 않으면 `comp` 전체에 `transform` 적용.

---

### Companion object

| 항목 | 값 / 설명 |
|---|---|
| `logger` | `Logger.getLogger(StreamingParser::class.java.name)` |
| `KEY_MATCH_REGEX` | `Regex("\"([^\"]+)\"\\s*:\\s*$")` — `fixJson`에서 마지막 키 추출 |
| `PREV_KEY_MATCHES_REGEX` | `Regex("\"key\"\\s*:\\s*\"([^\"]+)\"")` — `fixJson`에서 이전 `"key"` 값 찾기 |
| `SURFACE_ID_REGEX` | `Regex("\"surfaceId\"\\s*:\\s*\"([^\"]+)\"")` |
| `ROOT_ID_REGEX` | `Regex("\"root\"\\s*:\\s*\"([^\"]+)\"")` |
| `LATEST_VALUE_REGEX_CACHE` | `mutableMapOf<String, Regex>()` — 키별 정규식 캐시 |
| `MAX_JSON_BUFFER_SIZE` | `5 * 1024 * 1024` (5 MB) — JSON 버퍼 최대 크기 |

#### 팩토리 함수 `create(catalog, schemaMappings): StreamingParser`

`catalog?.version == A2uiVersion.VERSION_0_9`이면 `StreamingParserV09` 반환, 그렇지 않으면 `StreamingParserV08` 반환.

---

## 동작 흐름

1. **스트림 입력:** `processChunk(chunk)` 가 청크를 수신해 `buffer`에 쌓는다.
2. **구분자 탐지:** `<a2ui>` 태그 이전의 텍스트는 `ResponsePart(text=...)` 로 즉시 방출; 태그 발견 시 JSON 파싱 모드 진입.
3. **문자 단위 파싱:** `processJsonChunk`가 JSON 문자들을 `jsonBuffer`에 누적하면서 `{}`/`[]` 중첩을 `braceStack`으로 추적한다. 완전한 `{...}` 블록이 닫힐 때마다 파싱을 시도한다.
4. **완성된 객체 처리:** 컴포넌트(`id`+`component` 키 존재)는 `handlePartialComponent`로 `seenComponents`에 등록. 프로토콜 메시지는 `handleCompleteObject`(서브클래스 구현)로 처리.
5. **스니핑:** 각 메타 문자(`"`, `:`, `,`, `}`, `]`)마다 `sniffMetadata`를 호출해 `surfaceId`, `rootId` 등을 미리 파악한다. 청크 말미에 `sniffPartialComponent`, `sniffPartialDataModel`로 아직 닫히지 않은 부분 객체에서도 정보를 추출한다.
6. **위상 방출:** `topologyDirty == true`이거나 블록 종료 시 `yieldReachable`이 호출된다. `TopologyAnalyzer`로 root에서 도달 가능한 컴포넌트를 결정하고, 미수신 자식은 플레이스홀더로 대체한 뒤 `constructPartialMessage`로 부분 UI 메시지를 방출한다.
7. **`</a2ui>` 태그:** `processJsonChunk` 최종 호출 후 `resetJsonState`로 상태 초기화; 유효한 JSON이 없으면 예외 발생.
8. **중복 제거:** `processChunk` 반환 직전에 동일 `surfaceId`의 중복 `surfaceUpdate` 메시지를 제거한다.
