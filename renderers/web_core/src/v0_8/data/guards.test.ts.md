# renderers/web_core/src/v0_8/data/guards.test.ts

## 개요

`guards.ts`에서 내보낸 타입 가드 함수들의 단위 테스트 파일이다. Node.js 내장 `node:test` 러너와 `node:assert`를 사용한다. 기본 유틸리티 가드, 모든 `isResolved*` 컴포넌트 가드, 그리고 비공개 내부 가드를 공개 가드를 통해 간접 검증하는 케이스로 구성된다.

## 의존성

### 저장소 내부 모듈
- [`./guards.js`](./guards.ts.md) — 테스트 대상 모듈 (`.js` 확장자는 ESM 런타임 해석용)

### 외부 패키지
- `node:assert` — `strictEqual` 등 단언 함수
- `node:test` — `describe`, `it` 테스트 러너

## Exports

없음 (테스트 파일)

## 테스트 케이스 명세

### 공통 픽스처

파일 최상위 `describe('v0.8 Guards')` 블록 내에 다음 픽스처를 정의한다:
- `validComponentNode` — `{id: '1', type: 'Text', properties: {}}` — `isAnyComponentNode`를 통과하는 최소 노드
- `validStringValue` — `{literal: 'hello'}` — 문자열 리터럴 형태의 `StringValue`
- `validNumberValue` — `{literal: 42}` — 숫자 리터럴 형태의 `NumberValue`
- `validBooleanValue` — `{literal: true}` — 불리언 리터럴 형태의 `BooleanValue`

---

### `describe('Basics')`

#### `it('isValueMap')`
- `{key: 'k1', valueString: 'v1'}` → `true` (key 프로퍼티 존재)
- `{notKey: 'k1'}` → `false` (key 없음)
- `null` → `false`
- `'string'` → `false`

#### `it('isPath')`
- `isPath('path', '/a/b')` → `true` (키가 `'path'`이고 문자열)
- `isPath('path', 123)` → `false` (값이 문자열이 아님)
- `isPath('notPath', '/a/b')` → `false` (키가 `'path'`가 아님)

#### `it('isObject')`
- `{}` → `true`
- `[]` → `false` (배열 제외)
- `null` → `false`
- `42` → `false`

#### `it('isComponentArrayReference')`
- `{explicitList: ['1', '2']}` → `true`
- `{template: {}}` → `true`
- `{}` → `false` (두 키 모두 없음)
- `null` → `false`

---

### `describe('Component Resolution Guards')`

각 `it` 케이스는 해당 가드의 필수 프로퍼티가 존재/부재할 때의 동작을 검증한다.

#### `it('isResolvedAudioPlayer')`
- `{url: validStringValue}` → `true`
- `{url: 42}` → `false` (url이 StringValue가 아님)
- `{}` → `false`

#### `it('isResolvedButton')`
- `{child: validComponentNode, action: {}}` → `true`
- `{child: validComponentNode}` (action 없음) → `false`
- `{action: {}}` (child 없음) → `false`
- `{}` → `false`

#### `it('isResolvedCard')`
- `{child: validComponentNode}` → `true`
- `{children: [validComponentNode]}` → `true`
- `{children: 'not array'}` → `false` (배열 아님)
- `{}` → `false`
- `null` → `false`

#### `it('isResolvedCheckbox')`
- `{label: validStringValue, value: validBooleanValue}` → `true`
- `{label: validStringValue}` (value 없음) → `false`
- `{value: validBooleanValue}` (label 없음) → `false`

#### `it('isResolvedColumn')`
- `{children: [validComponentNode]}` → `true`
- `{children: {}}` (배열 아님) → `false`
- `{}` → `false`

#### `it('isResolvedDateTimeInput')`
- `{value: validStringValue}` → `true`
- `{}` → `false`

#### `it('isResolvedDivider')`
- `{anyOptionalProp: true}` → `true` (객체이면 통과)
- `null` → `false`

#### `it('isResolvedImage')`
- `{url: validStringValue}` → `true`
- `{}` → `false`

#### `it('isResolvedIcon')`
- `{name: validStringValue}` → `true`
- `{}` → `false`

#### `it('isResolvedList')`
- `{children: [validComponentNode]}` → `true`
- `{children: {}}` → `false`
- `{}` → `false`

#### `it('isResolvedModal')`
- `{entryPointChild: validComponentNode, contentChild: validComponentNode}` → `true`
- `{entryPointChild: validComponentNode}` (contentChild 없음) → `false`

#### `it('isResolvedMultipleChoice')`
- `{selections: []}` → `true`
- `{}` → `false`

#### `it('isResolvedRow')`
- `{children: [validComponentNode]}` → `true`
- `{children: {}}` → `false`
- `{}` → `false`

#### `it('isResolvedSlider')`
- `{value: validNumberValue}` → `true`
- `{}` → `false`

#### `it('isResolvedTabs (and isResolvedTabItem)')`
- `{tabItems: [{title: validStringValue, child: validComponentNode}]}` → `true`
- `{tabItems: [{title: validStringValue}]}` (child 없음) → `false`; `isResolvedTabItem` 내부 검증을 간접 테스트
- `{tabItems: {}}` (배열 아님) → `false`
- `{}` → `false`

#### `it('isResolvedText')`
- `{text: validStringValue}` → `true`
- `{}` → `false`

#### `it('isResolvedTextField')`
- `{label: validStringValue}` → `true`
- `{}` → `false`

#### `it('isResolvedVideo')`
- `{url: validStringValue}` → `true`
- `{}` → `false`

---

### `describe('Internal/Private structural guards via components')`

비공개 내부 가드를 공개 가드를 통해 간접 검증한다.

#### `it('isStringValue via Text')`
`isResolvedText`를 통해 `isStringValue`의 세 형태를 검증:
- `{text: {path: '.'}}` → `true` (path 형태)
- `{text: {literalString: 'hello'}}` → `true` (literalString 형태)
- `{text: {invalid: 'string'}}` → `false` (유효한 키 없음)

#### `it('isNumberValue via Slider')`
`isResolvedSlider`를 통해 `isNumberValue`의 세 형태를 검증:
- `{value: {path: '.'}}` → `true`
- `{value: {literalNumber: 42}}` → `true`
- `{value: {invalid: 42}}` → `false`

#### `it('isBooleanValue via Checkbox')`
`isResolvedCheckbox`를 통해 `isBooleanValue`의 세 형태를 검증:
- `{label: validStringValue, value: {path: '.'}}` → `true`
- `{label: validStringValue, value: {literalBoolean: true}}` → `true`
- `{label: validStringValue, value: {invalid: true}}` → `false`

#### `it('isAnyComponentNode edge cases via Row')`
`isResolvedRow`를 통해 `isAnyComponentNode`의 경계 케이스 검증:
- `{children: [{id: '1', type: 'Text'}]}` → `false` (properties 키 없음)
- `{children: [null]}` → `false`
- `{children: ['string']}` → `false`

## 동작 흐름

테스트는 `node:test`의 `describe`/`it` 계층 구조로 그룹화된다. 각 테스트는 가드 함수에 유효/무효 입력을 직접 전달하고 `assert.strictEqual`로 불리언 결과를 검증한다. 모킹(mocking)이나 픽스처 설정/해제(`beforeEach`/`afterEach`)는 없으며, `describe` 블록 최상위에 공유 픽스처 상수를 선언해 여러 테스트에서 재사용한다.
