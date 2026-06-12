# specification/v0_8/eval/src/validator.ts

## 개요

A2UI 프로토콜 메시지의 구조적 유효성을 검증하는 모듈이다. 입력된 데이터 객체가 `surfaceUpdate`, `dataModelUpdate`, `beginRendering`, `deleteSurface` 중 어떤 메시지 유형인지 판별한 뒤, 각 유형에 맞는 전용 검증 함수로 위임하여 오류 목록을 수집한다. 선택적으로 외부에서 주입된 `SchemaMatcher` 배열을 통해 추가 검증을 수행할 수 있어, 평가(eval) 시나리오에서 확장 가능한 검증 파이프라인을 구성한다.

## 의존성

### 외부 패키지
없음.

### 저장소 내부 모듈
- [`./surface_update_schema_matcher`](./surface_update_schema_matcher.ts.md) — `SurfaceUpdateSchemaMatcher` 클래스 (import만 되어 있으며, 타입 참조 및 외부 matcher 주입에 활용)
- [`./schema_matcher`](./schema_matcher.ts.md) — `SchemaMatcher` 추상 클래스. `validate(data): ValidationResult` 계약을 정의

## Exports

| 이름 | 종류 |
|------|------|
| `validateSchema` | 함수 (공개) |

나머지 함수(`validateDeleteSurface`, `validateSurfaceUpdate`, `validateDataModelUpdate`, `validateBeginRendering`, `validateBoundValue`, `validateComponent`)는 모두 모듈-내부 비공개 함수다.

---

## 상세 명세

### `validateSchema(data, schemaName, matchers?): string[]`

**시그니처**
```
function validateSchema(
  data: any,
  schemaName: string,
  matchers?: SchemaMatcher[],
): string[]
```

**동작 로직**

1. 빈 `errors: string[]` 배열을 초기화한다.
2. `data`의 최상위 키를 순서대로 확인한다:
   - `data.surfaceUpdate` 가 존재하면 → `validateSurfaceUpdate(data.surfaceUpdate, errors)` 호출
   - `data.dataModelUpdate` 가 존재하면 → `validateDataModelUpdate(data.dataModelUpdate, errors)` 호출
   - `data.beginRendering` 가 존재하면 → `validateBeginRendering(data.beginRendering, errors)` 호출
   - `data.deleteSurface` 가 존재하면 → `validateDeleteSurface(data.deleteSurface, errors)` 호출
   - 해당 키가 없으면 `'A2UI Protocol message must have one of: surfaceUpdate, dataModelUpdate, beginRendering, deleteSurface.'` 오류를 추가한다.
3. `matchers` 배열이 제공된 경우, 각 `matcher`에 대해 `matcher.validate(data)`를 호출한다. 결과의 `success`가 `false`이면 `result.error`를 `errors`에 추가한다.
4. 최종 `errors` 배열을 반환한다. 오류가 없으면 빈 배열이다.

**경계 케이스**: `schemaName` 매개변수는 현재 함수 본문 내에서 직접 사용되지 않는다(시그니처에만 존재).

---

### `validateDeleteSurface(data, errors): void` (비공개)

**시그니처**
```
function validateDeleteSurface(data: any, errors: string[]): void
```

**동작 로직**

1. `data.surfaceId`가 `undefined`이면 `"DeleteSurface must have a 'surfaceId' property."` 오류를 추가한다.
2. 허용 키 목록은 `['surfaceId']`다. `data`의 모든 키를 순회하며 목록에 없는 키가 있으면 `` `DeleteSurface has unexpected property: ${key}` `` 오류를 추가한다.

---

### `validateSurfaceUpdate(data, errors): void` (비공개)

**시그니처**
```
function validateSurfaceUpdate(data: any, errors: string[]): void
```

**동작 로직**

1. `data.surfaceId`가 `undefined`이면 `"SurfaceUpdate must have a 'surfaceId' property."` 오류를 추가한다.
2. `data.components`가 없거나 배열이 아니면 `"SurfaceUpdate must have a 'components' array."` 오류를 추가하고 즉시 반환한다(early return).
3. `componentIds: Set<string>` 를 생성하고 배열을 순회하면서, `c.id`가 존재하면 Set 중복 체크를 수행한다. 이미 존재하는 ID면 `` `Duplicate component ID found: ${c.id}` `` 오류를 추가하고, 이후 Set에 추가한다.
4. 모든 컴포넌트에 대해 `validateComponent(component, componentIds, errors)`를 호출한다.

---

### `validateDataModelUpdate(data, errors): void` (비공개)

**시그니처**
```
function validateDataModelUpdate(data: any, errors: string[]): void
```

**동작 로직**

1. `data.surfaceId`가 `undefined`이면 `"DataModelUpdate must have a 'surfaceId' property."` 오류를 추가한다.
2. 허용 최상위 키 목록은 `['surfaceId', 'path', 'contents']`다. 목록에 없는 키가 있으면 `` `DataModelUpdate has unexpected property: ${key}` `` 오류를 추가한다.
3. `data.contents`가 배열이 아니면 `"DataModelUpdate must have a 'contents' array."` 오류를 추가하고 즉시 반환한다.
4. 내부 클로저 `validateValueProperty(item, itemErrors, prefix)`를 정의한다:
   - 인식하는 값 속성 목록: `['valueString', 'valueNumber', 'valueBoolean', 'valueMap']`
   - `item`에서 위 속성들의 존재 여부를 카운트하여 정확히 1개가 존재해야 한다. 아니면 `` `${prefix} must have exactly one value property (...), found ${valueCount}.` `` 오류를 추가하고 반환.
   - 유일한 값 속성이 `valueMap`인 경우:
     - `item.valueMap`이 배열이 아니면 `` `${prefix} 'valueMap' must be an array.` `` 오류를 추가하고 반환.
     - 각 `mapItem`에 대해:
       - `mapItem.key`가 없으면 `` `${prefix} 'valueMap' item at index ${index} is missing a 'key'.` `` 오류 추가.
       - 허용 값 속성 `['valueString', 'valueNumber', 'valueBoolean']` 중 정확히 1개가 있어야 한다. 아니면 오류 추가.
       - 허용 키 목록(`['key', 'valueString', 'valueNumber', 'valueBoolean']`) 이외의 키가 있으면 오류 추가.
5. `data.contents`를 forEach로 순회한다 (`item`, `index`):
   - `item.key`가 없으면 `` `DataModelUpdate 'contents' item at index ${index} is missing a 'key'.` `` 오류 추가.
   - `validateValueProperty(item, errors, ...)` 를 호출.
   - 허용 키 목록(`['key', 'valueString', 'valueNumber', 'valueBoolean', 'valueMap']`) 이외의 키가 있으면 오류 추가.

---

### `validateBeginRendering(data, errors): void` (비공개)

**시그니처**
```
function validateBeginRendering(data: any, errors: string[]): void
```

**동작 로직**

1. `data.surfaceId`가 `undefined`이면 `"BeginRendering message must have a 'surfaceId' property."` 오류를 추가한다.
2. `data.root`가 falsy이면 `"BeginRendering message must have a 'root' property."` 오류를 추가한다.

---

### `validateBoundValue(prop, propName, componentId, componentType, errors): void` (비공개)

**시그니처**
```
function validateBoundValue(
  prop: any,
  propName: string,
  componentId: string,
  componentType: string,
  errors: string[],
): void
```

**동작 로직**

BoundValue 구조체(`literalString | literalNumber | literalBoolean | path` 중 정확히 하나를 키로 갖는 객체)를 검증한다.

1. `prop`이 객체가 아니거나 `null`이거나 배열이면 `` `Component '${componentId}' of type '${componentType}' property '${propName}' must be an object.` `` 오류를 추가하고 반환.
2. `prop`의 모든 키를 추출하고, 허용 키 목록 `['literalString', 'literalNumber', 'literalBoolean', 'path']`에 속하는 키의 수(`validKeyCount`)를 센다.
3. `validKeyCount !== 1` 이거나 전체 키 개수(`keys.length`)가 1이 아닌 경우, 즉 키가 정확히 하나이고 그것이 허용 목록에 있어야 통과하는 구조다. 아니면 `` `Component '${componentId}' of type '${componentType}' property '${propName}' must have exactly one key from [...]. Found: ${keys.join(', ')}` `` 오류를 추가한다.

---

### `validateComponent(component, allIds, errors): void` (비공개)

**시그니처**
```
function validateComponent(component: any, allIds: Set<string>, errors: string[]): void
```

**동작 로직**

개별 컴포넌트 객체를 검증한다.

**전처리 (공통 구조 검증)**

1. `component.id`가 없으면 `"Component is missing an 'id'."` 오류를 추가하고 반환.
2. `component.component`가 없으면 `` `Component '${component.id}' is missing 'component'.` `` 오류를 추가하고 반환.
3. `component.component`의 키 목록(`componentTypes`)을 추출한다. 키가 정확히 1개가 아니면 오류를 추가하고 반환.
4. `componentType = componentTypes[0]`, `properties = component.component[componentType]` 으로 설정한다.

**내부 헬퍼 클로저**

- `checkRequired(props: string[])`: `props` 배열의 각 항목이 `properties`에 정의되어 있지 않으면 `` `Component '...' of type '...' is missing required property '${prop}'.` `` 오류 추가.
- `checkRefs(ids: (string | undefined)[])`: 각 `id`가 truthy이면서 `allIds` Set에 없으면 `` `Component '...' references non-existent component ID '${id}'.` `` 오류 추가.

**컴포넌트 타입별 switch 분기**

| `componentType` | 필수 속성 | BoundValue 검증 대상 | 추가 검증 |
|---|---|---|---|
| `Heading` | `text` | `text` | — |
| `Text` | `text` | `text` | — |
| `Image` | `url` | `url` | — |
| `Video` | `url` | `url` | — |
| `AudioPlayer` | `url` | `url`, 선택적으로 `description` | — |
| `TextField` | `label` | `label`, 선택적으로 `text` | — |
| `DateTimeInput` | `value` | `value` | — |
| `MultipleChoice` | `selections`, `options` | 각 옵션의 `label` | `selections`는 `literalArray` 또는 `path` 중 하나를 가져야 함. `options`가 배열이면 각 항목에 `label`, `value` 필드 확인 |
| `Slider` | `value` | `value` | — |
| `CheckBox` | `value`, `label` | `value`, `label` | — |
| `Row` / `Column` / `List` | `children` | — | `children`이 배열이면 `explicitList` 또는 `template` 중 정확히 하나를 가져야 함. `explicitList` 존재 시 해당 ID 참조 검증, `template` 존재 시 `template.componentId` 참조 검증 |
| `Card` | `child` | — | `properties.child` ID 참조 검증 |
| `Tabs` | `tabItems` | 각 탭의 `title` | `tabItems` 배열 순회: 각 `tab`에 `title`, `child` 필수. `tab.child` ID 참조 검증 |
| `Modal` | `entryPointChild`, `contentChild` | — | 두 필드 모두 ID 참조 검증 |
| `Button` | `child`, `action` | — | `properties.child` ID 참조 검증. `action.name`이 없으면 오류 추가 |
| `Divider` | 없음 | — | — |
| `Icon` | `name` | `name` | — |
| 알 수 없는 타입 | — | — | `` `Unknown component type '${componentType}' in component '${component.id}'.` `` 오류 추가 |

**`Row`/`Column`/`List`의 children 검증 상세**: `properties.children`이 배열 타입이더라도 실제로는 객체 형태를 기대하는 구조임에 주의. `Array.isArray(properties.children)` 조건이 참일 때만 내부 검증이 수행되며, `explicitList`와 `template` 속성은 배열이 아닌 객체 프로퍼티로 읽힌다.

---

## 동작 흐름

`validateSchema`가 진입점이며, 아래 순서로 제어가 흐른다:

1. **메시지 유형 판별** → 최상위 키(`surfaceUpdate` 등)로 분기
2. **유형별 검증 함수 위임** → 각 함수가 `errors` 배열에 오류를 직접 append (변이)
3. **컴포넌트 트리 검증** (`surfaceUpdate`인 경우): `validateSurfaceUpdate` → 각 컴포넌트 → `validateComponent` → 각 속성 → `validateBoundValue`
4. **외부 matcher 실행**: 제공된 `matchers` 배열 순회, 각 `matcher.validate(data)` 호출 후 실패 결과 병합
5. **결과 반환**: 축적된 `errors` 배열 반환. 빈 배열은 유효함을 의미

데이터 흐름 상 `errors` 배열은 단일 참조로 공유되어 모든 하위 검증 함수에서 mutate된다. 각 유형 검증 함수는 반환값 없이 side-effect로 오류를 추가한다.
