# renderers/web_core/src/v0_8/data/guards.ts

## 개요

v0.8 데이터 모델의 타입 가드(type guard) 함수 모음이다. `unknown` 타입의 값을 받아 특정 v0.8 타입에 해당하는지 런타임에 검사하고, TypeScript의 타입 좁히기(narrowing)를 제공한다. 기본 유틸리티 가드부터 모든 해석된(resolved) 컴포넌트 props 타입까지 망라한다.

## 의존성

### 저장소 내부 모듈
- [`../types/primitives`](../types/primitives.ts.md) — `BooleanValue`, `NumberValue`, `StringValue` 타입
- [`../types/types`](../types/types.ts.md) — `AnyComponentNode`, `ComponentArrayReference`, `ResolvedAudioPlayer`, `ResolvedButton`, `ResolvedCard`, `ResolvedCheckbox`, `ResolvedColumn`, `ResolvedDateTimeInput`, `ResolvedDivider`, `ResolvedIcon`, `ResolvedImage`, `ResolvedList`, `ResolvedModal`, `ResolvedMultipleChoice`, `ResolvedRow`, `ResolvedSlider`, `ResolvedTabItem`, `ResolvedTabs`, `ResolvedText`, `ResolvedTextField`, `ResolvedVideo`, `ValueMap` 타입

### 외부 패키지
없음

## Exports

| 이름 | 종류 | 설명 |
|---|---|---|
| `isValueMap` | 함수 (타입 가드) | `ValueMap` 여부 검사 |
| `isPath` | 함수 (타입 가드) | 키가 `'path'`이고 값이 문자열인지 검사 |
| `isObject` | 함수 (타입 가드) | null이 아닌 비배열 객체인지 검사 |
| `isComponentArrayReference` | 함수 (타입 가드) | `ComponentArrayReference` 여부 검사 |
| `isResolvedAudioPlayer` | 함수 (타입 가드) | `ResolvedAudioPlayer` props 여부 |
| `isResolvedButton` | 함수 (타입 가드) | `ResolvedButton` props 여부 |
| `isResolvedCard` | 함수 (타입 가드) | `ResolvedCard` props 여부 |
| `isResolvedCheckbox` | 함수 (타입 가드) | `ResolvedCheckbox` props 여부 |
| `isResolvedColumn` | 함수 (타입 가드) | `ResolvedColumn` props 여부 |
| `isResolvedDateTimeInput` | 함수 (타입 가드) | `ResolvedDateTimeInput` props 여부 |
| `isResolvedDivider` | 함수 (타입 가드) | `ResolvedDivider` props 여부 |
| `isResolvedImage` | 함수 (타입 가드) | `ResolvedImage` props 여부 |
| `isResolvedIcon` | 함수 (타입 가드) | `ResolvedIcon` props 여부 |
| `isResolvedList` | 함수 (타입 가드) | `ResolvedList` props 여부 |
| `isResolvedModal` | 함수 (타입 가드) | `ResolvedModal` props 여부 |
| `isResolvedMultipleChoice` | 함수 (타입 가드) | `ResolvedMultipleChoice` props 여부 |
| `isResolvedRow` | 함수 (타입 가드) | `ResolvedRow` props 여부 |
| `isResolvedSlider` | 함수 (타입 가드) | `ResolvedSlider` props 여부 |
| `isResolvedTabs` | 함수 (타입 가드) | `ResolvedTabs` props 여부 |
| `isResolvedText` | 함수 (타입 가드) | `ResolvedText` props 여부 |
| `isResolvedTextField` | 함수 (타입 가드) | `ResolvedTextField` props 여부 |
| `isResolvedVideo` | 함수 (타입 가드) | `ResolvedVideo` props 여부 |

## 상세 명세

### 기본 유틸리티 가드 (public)

#### `isObject(value: unknown): value is Record<string, unknown>`
값이 `typeof === 'object'`이고, `null`이 아니며, 배열이 아닐 때 `true`를 반환한다. 모든 다른 가드의 기반이 되는 가장 원시적인 검사다.

#### `isValueMap(value: unknown): value is ValueMap`
`isObject(value)`를 통과하고, 추가로 `'key'` 프로퍼티가 존재하는지(`'key' in value`) 확인한다.

#### `isPath(key: string, value: unknown): value is string`
`key === 'path'`이고 `typeof value === 'string'`인 경우에만 `true`를 반환한다. 두 조건을 모두 만족해야 하므로, 키 이름과 값의 타입을 동시에 검사한다.

#### `isComponentArrayReference(value: unknown): value is ComponentArrayReference`
`isObject` 검사를 통과한 후, `'explicitList'` 또는 `'template'` 키 중 하나라도 존재하면 `true`를 반환한다.

---

### 내부 값 타입 가드 (private, 비공개)

#### `isStringValue(value: unknown): value is StringValue`
세 가지 형태 중 하나를 만족해야 한다:
- `'path' in value` (경로 참조 형태)
- `'literal' in value && typeof value.literal === 'string'` (문자열 리터럴 형태)
- `'literalString' in value` (대안적 문자열 리터럴 키 형태)

#### `isNumberValue(value: unknown): value is NumberValue`
세 가지 형태 중 하나를 만족해야 한다:
- `'path' in value`
- `'literal' in value && typeof value.literal === 'number'`
- `'literalNumber' in value`

#### `isBooleanValue(value: unknown): value is BooleanValue`
세 가지 형태 중 하나를 만족해야 한다:
- `'path' in value`
- `'literal' in value && typeof value.literal === 'boolean'`
- `'literalBoolean' in value`

#### `isAnyComponentNode(value: unknown): value is AnyComponentNode`
`isObject` 통과 후, `'id'`, `'type'`, `'properties'` 세 키가 모두 존재하는지 확인한다. 세 키 중 하나라도 없으면 `false`를 반환한다.

---

### 내부 복합 가드 (private)

#### `isResolvedTabItem(item: unknown): item is ResolvedTabItem`
`isObject` 통과 후 다음을 모두 확인한다:
- `'title' in item && isStringValue(item.title)`
- `'child' in item && isAnyComponentNode(item.child)`

---

### 컴포넌트 props 가드 (public)

모든 `isResolved*` 함수는 `props: unknown`을 받아 해당 `Resolved*` 타입인지 반환한다.

#### `isResolvedAudioPlayer`
`isObject` + `'url' in props` + `isStringValue(props.url)`

#### `isResolvedButton`
`isObject` + `'child' in props` + `isAnyComponentNode(props.child)` + `'action' in props`

#### `isResolvedCard`
복잡한 분기를 가진다:
1. `isObject` 실패 시 `false`
2. `'child' in props`가 없으면 `'children' in props` 확인
   - `children`도 없으면 `false`
   - `children`이 있으면 `Array.isArray && .every(isAnyComponentNode)`
3. `'child'`가 있으면 `isAnyComponentNode(props.child)` 검사

#### `isResolvedCheckbox`
`isObject` + `'label' in props` + `isStringValue(props.label)` + `'value' in props` + `isBooleanValue(props.value)` 모두 만족해야 한다.

#### `isResolvedColumn`
`isObject` + `'children' in props` + `Array.isArray(props.children)` + `.every(isAnyComponentNode)`

#### `isResolvedDateTimeInput`
`isObject` + `'value' in props` + `isStringValue(props.value)`

#### `isResolvedDivider`
`isObject(props)`만 확인한다. Divider는 모든 프로퍼티가 선택적이므로 객체인지만 검사하면 충분하다.

#### `isResolvedImage`
`isObject` + `'url' in props` + `isStringValue(props.url)`

#### `isResolvedIcon`
`isObject` + `'name' in props` + `isStringValue(props.name)`

#### `isResolvedList`
`isObject` + `'children' in props` + `Array.isArray(props.children)` + `.every(isAnyComponentNode)`

#### `isResolvedModal`
`isObject` + `'entryPointChild' in props` + `isAnyComponentNode(props.entryPointChild)` + `'contentChild' in props` + `isAnyComponentNode(props.contentChild)` 모두 필요

#### `isResolvedMultipleChoice`
`isObject` + `'selections' in props` (selections 값의 타입은 추가 검증 없음)

#### `isResolvedRow`
`isObject` + `'children' in props` + `Array.isArray(props.children)` + `.every(isAnyComponentNode)`

#### `isResolvedSlider`
`isObject` + `'value' in props` + `isNumberValue(props.value)`

#### `isResolvedTabs`
`isObject` + `'tabItems' in props` + `Array.isArray(props.tabItems)` + `.every(isResolvedTabItem)`

#### `isResolvedText`
`isObject` + `'text' in props` + `isStringValue(props.text)`

#### `isResolvedTextField`
`isObject` + `'label' in props` + `isStringValue(props.label)`

#### `isResolvedVideo`
`isObject` + `'url' in props` + `isStringValue(props.url)`

## 동작 흐름

이 파일의 모든 함수는 순수 함수(side-effect 없음)이며 상태를 유지하지 않는다. 호출자는 `unknown` 타입의 값을 넘기고, 가드 함수는 불리언을 반환하며 TypeScript 컴파일러에게 타입 좁히기 정보를 제공한다. `isObject`를 최하위 공통 기반으로 하여, 내부 값 타입 가드(`isStringValue` 등)와 컴포넌트 가드가 계층적으로 조합된다. `isResolvedCard`만 `child`/`children` 두 경로를 분기 처리하는 특수 로직을 가진다.
