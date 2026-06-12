# renderers/web_core/src/v0_8/data/model-processor.ts

## 개요

`A2uiMessageProcessor` 클래스를 정의하며, 서버로부터 수신한 `ServerToClientMessage` 배열을 처리해 UI 서페이스(Surface) 집합을 구조적으로 유지·갱신하는 역할을 한다. 메시지 종류(시작 렌더링, 서페이스 업데이트, 데이터 모델 갱신, 서페이스 삭제)에 따라 내부 상태를 변이시키고, 컴포넌트 속성 및 데이터 바인딩을 재귀적으로 해결해 `AnyComponentNode` 트리를 재구성한다. `MessageProcessor` 인터페이스를 구현함으로써 외부 렌더러가 일관된 API로 모델을 소비할 수 있게 한다.

## 의존성

### 외부 패키지
- (없음 — 모든 의존성은 저장소 내부 모듈)

### 저장소 내부 모듈
- [`../types/types.js`](../types/types.ts.md) — `ServerToClientMessage`, `AnyComponentNode`, `BeginRenderingMessage`, `DataArray`, `DataMap`, `DataModelUpdate`, `DataValue`, `DeleteSurfaceMessage`, `ResolvedMap`, `ResolvedValue`, `Surface`, `SurfaceID`, `SurfaceUpdateMessage`, `MessageProcessor`, `ValueMap`
- [`../schema/server-to-client.js`](../schema/server-to-client.ts.md) — `A2uiMessageSchema`
- [`../errors.js`](../errors.ts.md) — `A2uiStateError`, `A2uiValidationError`
- [`./guards.js`](./guards.ts.md) — `isComponentArrayReference`, `isObject`, `isPath`, `isResolvedAudioPlayer`, `isResolvedButton`, `isResolvedCard`, `isResolvedCheckbox`, `isResolvedColumn`, `isResolvedDateTimeInput`, `isResolvedDivider`, `isResolvedIcon`, `isResolvedImage`, `isResolvedList`, `isResolvedModal`, `isResolvedMultipleChoice`, `isResolvedRow`, `isResolvedSlider`, `isResolvedTabs`, `isResolvedText`, `isResolvedTextField`, `isResolvedVideo`

## Exports

| 이름 | 종류 |
|------|------|
| `A2uiMessageProcessor` | 클래스 |

## 상세 명세

### 클래스 `A2uiMessageProcessor`

`MessageProcessor` 인터페이스를 구현하는 메인 클래스.

#### 정적 상수

- `static readonly DEFAULT_SURFACE_ID = '@default'` — surfaceId가 명시되지 않았을 때 사용하는 기본 서페이스 식별자.

#### 인스턴스 필드

| 필드 | 타입 | 초기값 | 설명 |
|------|------|--------|------|
| `mapCtor` | `MapConstructor` | `Map` | `Map` 인스턴스 생성에 사용할 생성자 |
| `arrayCtor` | `ArrayConstructor` | `Array` | 배열 생성에 사용할 생성자 |
| `setCtor` | `SetConstructor` | `Set` | `Set` 생성에 사용할 생성자 |
| `objCtor` | `ObjectConstructor` | `Object` | 플레인 객체 생성에 사용할 생성자 |
| `surfaces` | `Map<SurfaceID, Surface>` | `new opts.mapCtor()` | 모든 서페이스를 보관하는 맵 |

#### 생성자

```ts
constructor(opts?: { mapCtor, arrayCtor, setCtor, objCtor })
```

- `opts`의 각 생성자를 인스턴스 필드에 할당하고, `opts.mapCtor`로 `surfaces` 맵을 초기화한다.
- `opts`를 생략하면 표준 `Map`, `Array`, `Set`, `Object`가 사용된다.
- 이 설계는 커스텀 컨테이너 구현체를 주입할 수 있도록 하며, 테스트 환경이나 특수 런타임에서 활용된다.

---

#### `getSurfaces(): ReadonlyMap<string, Surface>`

- 내부 `surfaces` 맵 전체를 순회해, `rootComponentId`가 존재하는(truthy) 서페이스만 새 `Map`에 담아 반환한다.
- `rootComponentId`가 없는 서페이스(예: `beginRendering` 이전에 `surfaceUpdate`만 수신된 경우)는 존재하지만 렌더 시 오류가 발생하므로 노출하지 않는다.
- 반환값은 읽기 전용(`ReadonlyMap`)이다.

---

#### `clearSurfaces(): void`

- 내부 `surfaces` 맵을 완전히 비운다(`Map.clear()`).

---

#### `processMessages(messages: ServerToClientMessage[]): void`

- 배열을 순서대로 순회하며 각 `rawMessage`를 `A2uiMessageSchema.parse(rawMessage)`로 검증/파싱한다.
- 파싱된 메시지에서 다음 필드를 순서대로 확인하고 존재하면 대응 핸들러를 호출한다:
  1. `message.beginRendering` → `handleBeginRendering`
  2. `message.surfaceUpdate` → `handleSurfaceUpdate`
  3. `message.dataModelUpdate` → `handleDataModelUpdate`
  4. `message.deleteSurface` → `handleDeleteSurface`
- 각 핸들러는 메시지 내부의 `surfaceId`를 사용한다.

---

#### `getData(node: AnyComponentNode, relativePath: string, surfaceId?: string): DataValue | null`

- `surfaceId`의 기본값은 `DEFAULT_SURFACE_ID`이다.
- `getOrCreateSurface`로 서페이스를 가져온다; 없으면 `null` 반환.
- `relativePath`가 `'.'` 또는 `''`이면 `finalPath`를 `node.dataContextPath ?? '/'`로 설정한다.
- 그 외에는 `resolvePath(relativePath, node.dataContextPath)`로 절대 경로를 계산한다.
- `getDataByPath(surface.dataModel, finalPath)`로 최종 값을 조회해 반환한다.

---

#### `setData(node: AnyComponentNode | null, relativePath: string, value: DataValue, surfaceId?: string): void`

- `node`가 `null`이면 `console.warn`을 출력하고 즉시 반환한다.
- `surfaceId`의 기본값은 `DEFAULT_SURFACE_ID`이다.
- `getData`와 동일한 경로 해결 로직을 거친 뒤 `setDataByPath(surface.dataModel, finalPath, value)`를 호출한다.

---

#### `resolvePath(path: string, dataContextPath?: string): string`

- `path`가 `'/'`로 시작하면 절대 경로로 판단해 `path`를 그대로 반환한다.
- `dataContextPath`가 존재하고 `'/'`가 아니면, 두 경로 사이에 정확히 하나의 슬래시만 생기도록 결합해 반환한다.
- 그 외(컨텍스트 없음 또는 루트 컨텍스트)에는 `'/' + path`를 반환한다.

---

#### `private parseIfJsonString(value: DataValue): DataValue`

- `value`가 문자열이 아니면 즉시 반환한다.
- 문자열을 trim한 뒤 `{...}` 또는 `[...]` 형태면 `JSON.parse`를 시도한다.
  - 파싱 성공 시 파싱된 값을 반환한다.
  - 파싱 실패 시 `console.warn`을 출력하고 원본 문자열을 반환한다.
- JSON-like하지 않은 문자열은 그대로 반환한다.

---

#### `private convertKeyValueArrayToMap(arr: DataArray): DataMap`

- 특정 키-값 배열 형식(`[{ key: "...", value_*: ... }, ...]`)을 `DataMap`(`Map<string, DataValue>`)으로 변환한다.
- 각 `item`이 객체이고 `key` 프로퍼티를 가질 때만 처리한다.
- `findValueKey(item)`으로 `value`로 시작하는 프로퍼티 키를 찾는다. 없으면 건너뛴다.
- `valueKey === 'valueMap'`이고 값이 배열이면 재귀적으로 `convertKeyValueArrayToMap`을 호출한다.
- 문자열 값이면 `parseIfJsonString`을 통해 JSON 파싱을 시도한다.
- 최종 값을 `setDataByPath(map, key, value)`로 맵에 삽입한다.

---

#### `private setDataByPath(root: DataMap, path: string, value: DataValue | ValueMap[]): void`

배열 형태의 특수 컨벤션 처리:
1. `value`가 배열이고, 비어있거나 첫 원소가 `key` 프로퍼티를 가진 객체이면, 키-값 배열로 판단한다.
   - 길이가 1이고 `key === '.'`인 경우: "경로에 원시값 설정" 컨벤션으로, `findValueKey`로 값을 추출하고 필요시 `valueMap` 재귀 변환 또는 JSON 파싱을 수행한다.
   - 그 외: `convertKeyValueArrayToMap`으로 맵 변환한다.

경로 탐색:
2. `normalizePath(path)`로 경로를 정규화하고 `'/'`로 분리한 뒤 빈 세그먼트를 제거한다.
3. 세그먼트가 없으면(루트 설정):
   - `value`가 `Map`이나 객체이면 루트 맵을 지우고 항목들을 복사한다. 일반 객체면 `Map`으로 변환한다.
   - 그 외에는 `console.error`를 출력한다.
4. 세그먼트가 있으면 마지막 세그먼트 직전까지 탐색하며, 중간 노드가 없으면 새 `mapCtor` 인스턴스를 생성해 삽입한다. 현재 노드가 `Map`이면 `get/set`, 배열이고 세그먼트가 숫자이면 인덱스 접근을 사용한다.
5. 최종 세그먼트에 `value`를 설정한다.

---

#### `private normalizePath(path: string): string`

두 단계 변환으로 경로를 슬래시 구분 형식으로 정규화한다:
1. `[index]` 괄호 표기를 `.index` 점 표기로 변환 (예: `[0]` → `.0`).
2. `.`으로 분리한 뒤 빈 세그먼트를 제거하고 `'/'` + join으로 절대 경로 형식을 만든다.
   - 예: `"bookRecommendations[0].title"` → `"/bookRecommendations/0/title"`
   - 예: `"book.0.title"` → `"/book/0/title"`

---

#### `private getDataByPath(root: DataMap, path: string): DataValue | null`

- `normalizePath(path)`로 정규화 후 세그먼트 목록을 만든다.
- 세그먼트를 순서대로 탐색한다:
  - 현재 값이 `null`/`undefined`면 `null` 반환.
  - `Map`이면 `get(segment)`.
  - 배열이고 숫자 세그먼트면 인덱스 접근.
  - 일반 객체면 프로퍼티 접근.
  - 위 조건 모두 해당하지 않으면 더 깊이 탐색할 수 없으므로 `null` 반환.
- 최종 도달한 값을 반환한다.

---

#### `private getOrCreateSurface(surfaceId: string): Surface`

- `surfaces.get(surfaceId)`로 기존 서페이스를 조회한다.
- 없으면 `objCtor`로 새 `Surface` 객체(`{ rootComponentId: null, componentTree: null, dataModel: new mapCtor(), components: new mapCtor(), styles: new objCtor() }`)를 생성하고 `surfaces.set`으로 등록한 뒤 반환한다.

---

#### `private handleBeginRendering(message: BeginRenderingMessage, surfaceId: SurfaceID): void`

- `getOrCreateSurface`로 서페이스 획득.
- `surface.rootComponentId = message.root` 설정.
- `surface.styles = message.styles ?? {}` 설정.
- `rebuildComponentTree(surface)` 호출.

---

#### `private handleSurfaceUpdate(message: SurfaceUpdateMessage, surfaceId: SurfaceID): void`

- `getOrCreateSurface`로 서페이스 획득.
- `message.components` 배열을 순회해 각 컴포넌트를 `surface.components.set(component.id, component)`로 등록한다.
- `rebuildComponentTree(surface)` 호출.

---

#### `private handleDataModelUpdate(message: DataModelUpdate, surfaceId: SurfaceID): void`

- `getOrCreateSurface`로 서페이스 획득.
- `path = message.path ?? '/'`.
- `setDataByPath(surface.dataModel, path, message.contents)`로 데이터 모델 갱신.
- `rebuildComponentTree(surface)` 호출.

---

#### `private handleDeleteSurface(message: DeleteSurfaceMessage): void`

- `surfaces.delete(message.surfaceId)`로 서페이스를 제거한다.

---

#### `private rebuildComponentTree(surface: Surface): void`

- `surface.rootComponentId`가 없으면 `surface.componentTree = null`로 설정하고 반환한다.
- 순환 참조 추적용 `Set<string>` (`visited`)을 생성한다.
- `buildNodeRecursive(surface.rootComponentId, surface, visited, '/', '')`를 호출해 루트부터 전체 트리를 재구성하고 `surface.componentTree`에 할당한다.

---

#### `private findValueKey(value: Record<string, unknown>): string | undefined`

- `Object.keys(value)`를 순회해 `'value'`로 시작하는 첫 번째 키를 반환한다.

---

#### `private buildNodeRecursive(baseComponentId: string, surface: Surface, visited: Set<string>, dataContextPath: string, idSuffix?: string): AnyComponentNode | null`

- `fullId = baseComponentId + idSuffix`를 생성한다.
- `components.has(baseComponentId)`가 없으면 `null` 반환.
- `visited.has(fullId)`이면 `A2uiStateError`를 던진다(순환 참조).
- `visited.add(fullId)` 후 처리를 시작한다.
- `componentData.component`에서 컴포넌트 타입명(`componentType`)과 미해결 속성(`unresolvedProperties`)을 추출한다.
- `resolvePropertyValue`를 각 속성에 호출해 `resolvedProperties: ResolvedMap`을 채운다.
- `visited.delete(fullId)` (현재 노드 처리 완료 표시).
- `baseNode = { id: fullId, dataContextPath, weight: componentData.weight ?? 'initial' }`를 구성한다.
- `componentType`에 따라 `switch`문으로 분기:
  - `'Text'`, `'Image'`, `'Icon'`, `'Video'`, `'AudioPlayer'`, `'Row'`, `'Column'`, `'List'`, `'Card'`, `'Tabs'`, `'Divider'`, `'Modal'`, `'Button'`, `'CheckBox'`, `'TextField'`, `'DateTimeInput'`, `'MultipleChoice'`, `'Slider'` — 각각 대응하는 `isResolved*` 가드를 통과하지 못하면 `A2uiValidationError`를 던지고, 통과하면 `{ ...baseNode, type: componentType, properties: resolvedProperties }` 노드를 `objCtor`로 생성해 반환한다.
  - `default` — 가드 없이 동일한 구조의 노드를 반환한다.

---

#### `private resolvePropertyValue(value: unknown, surface: Surface, visited: Set<string>, dataContextPath: string, idSuffix?: string): ResolvedValue`

단계별로 값의 종류를 판별해 처리한다:

1. **컴포넌트 ID 문자열**: `typeof value === 'string'`이고 `surface.components.has(value)`이면 `buildNodeRecursive`를 재귀 호출한다.
2. **`ComponentArrayReference`**: `isComponentArrayReference(value)`가 참이면:
   - `explicitList`가 있으면 해당 ID 배열을 `buildNodeRecursive`로 매핑해 노드 배열을 반환한다.
   - `template`이 있으면 `resolvePath(value.template.dataBinding, dataContextPath)`로 데이터 경로를 해결하고 `getDataByPath`로 데이터를 가져온다:
     - 데이터가 배열이면: 각 인덱스에 대해 부모 인덱스 경로에 현재 인덱스를 추가해 `:idx1:idx2` 형식의 `newSuffix`를 만들고 `buildNodeRecursive`를 호출한다.
     - 데이터가 `Map`이면: 각 키에 대해 `:key` 형식의 `newSuffix`를 만들고 `buildNodeRecursive`를 호출한다.
     - 데이터가 아직 없으면 빈 배열을 반환한다.
3. **일반 배열**: 각 항목에 `resolvePropertyValue`를 재귀 적용한다.
4. **일반 객체**: 새 `ResolvedMap`을 생성하고 각 프로퍼티를 재귀 처리한다. 단, 프로퍼티가 `isPath(key, propValue)`를 만족하고 `dataContextPath !== '/'`이면 경로에서 `/item`, `/text`, `/label`, 선행 `./` 또는 `/`를 제거해 상대 경로로 정리한다.
5. **원시값**: 값을 그대로 반환한다.

## 동작 흐름

1. 외부에서 `processMessages(messages)`를 호출하면 각 메시지를 스키마 검증 후 종류에 따라 핸들러로 분기한다.
2. `handleBeginRendering`은 서페이스에 루트 컴포넌트 ID와 스타일을 설정하고 트리를 재구성한다.
3. `handleSurfaceUpdate`는 컴포넌트 등록 후 트리를 재구성한다.
4. `handleDataModelUpdate`는 데이터 모델에 경로 기반으로 값을 쓰고 트리를 재구성한다.
5. `handleDeleteSurface`는 서페이스 자체를 제거한다.
6. 트리 재구성(`rebuildComponentTree`)은 항상 루트부터 시작해 재귀적으로 모든 컴포넌트 속성을 해결하며, 데이터 바인딩 템플릿은 현재 데이터 모델 상태에 따라 동적으로 펼쳐진다.
7. `getSurfaces()`를 통해 외부 렌더러가 완성된 `Surface` 맵을 소비한다.
