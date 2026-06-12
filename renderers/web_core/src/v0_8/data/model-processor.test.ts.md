# renderers/web_core/src/v0_8/data/model-processor.test.ts

## 개요

`A2uiMessageProcessor` 클래스의 단위 테스트 파일이다. 메시지 처리(`processMessages`), 데이터 모델 관리(`getData`/`setData`), 컴포넌트 트리 재구성(`rebuildComponentTree`), 템플릿 확장 등 `model-processor.ts`의 공개·비공개 동작을 포괄적으로 검증한다. Node.js 내장 `node:test` 러너와 `node:assert`를 사용한다.

## 의존성

### 저장소 내부 모듈
- [`./model-processor.js`](./model-processor.ts.md) — 테스트 대상 클래스 `A2uiMessageProcessor`
- [`../types/types.js`](../types/types.ts.md) — `TextNode`, `RowNode` 타입

### 외부 패키지
- `node:assert` — `ok`, `strictEqual`, `deepStrictEqual`, `throws`
- `node:test` — `describe`, `it`, `beforeEach`

## Exports

없음 (테스트 파일)

## 테스트 케이스 명세

### 공통 픽스처 및 설정

`beforeEach`에서 매 테스트 전에 `processor = new A2uiMessageProcessor()`로 새 인스턴스를 생성한다. 각 테스트는 독립된 `processor` 상태에서 시작한다.

---

### `it('handles beginRendering')`
**검증 동작:** `beginRendering` 메시지가 처리되면 surface가 생성되고 `rootComponentId`와 `styles`가 설정되며, 컴포넌트 없이는 `componentTree`가 `null`임을 확인한다.
- `processMessages`에 `{beginRendering: {surfaceId: 's1', root: 'root', styles: {font: 'Arial'}}}` 전달
- `getSurfaces().get('s1')`가 존재함
- `surface.rootComponentId === 'root'`
- `surface.styles` deepEqual `{font: 'Arial'}`
- `surface.componentTree === null`

### `it('handles surfaceUpdate')`
**검증 동작:** `beginRendering` 후 `surfaceUpdate`를 처리하면 `componentTree`가 구축되고 루트 노드의 타입, id, properties가 올바른지 확인한다.
- `beginRendering` 후 `surfaceUpdate`로 id=`'root'`, `Text` 컴포넌트 추가
- `surface.componentTree`가 존재하고, `root.id === 'root'`, `root.type === 'Text'`
- `root.properties.text` deepEqual `{literal: 'Hello'}`

### `it('handles dataModelUpdate')`
**검증 동작:** `dataModelUpdate` 메시지로 `contents`의 key-value 쌍이 `dataModel` Map에 올바르게 저장되는지 확인한다.
- `beginRendering` + `dataModelUpdate({key: 'message', valueString: 'World'})`
- `surface.dataModel.get('message') === 'World'`

### `it('handles deleteSurface')`
**검증 동작:** `deleteSurface` 메시지 처리 후 해당 surfaceId가 surfaces Map에서 제거되는지 확인한다.
- `beginRendering` → surfaces에 `'s1'` 존재 확인
- `deleteSurface({surfaceId: 's1'})` 처리
- `surfaces.has('s1') === false`

### `it('resolves component references (children)')`
**검증 동작:** `explicitList`로 참조된 자식 컴포넌트들이 올바르게 `RowNode`의 `children` 배열로 해석되는지 확인한다.
- `Row` 컴포넌트의 `children: {explicitList: ['t1', 't2']}`와 두 개의 `Text` 컴포넌트 등록
- `root.type === 'Row'`, `root.properties.children.length === 2`
- 각 자식의 `properties.text`가 `{literal: 'One'}`과 `{literal: 'Two'}` 임을 확인

### `it('resolves templates')`
**검증 동작:** 템플릿(`template.componentId` + `dataBinding`)으로 정의된 자식들이 데이터 모델의 Map 데이터를 기반으로 올바른 개수와 데이터 컨텍스트 경로를 가지고 확장되는지, 그리고 `getData`로 올바른 값을 해석하는지 확인한다.
- `items` key에 `{key: '0', valueString: 'Item A'}`, `{key: '1', valueString: 'Item B'}` 설정
- `Row` 컴포넌트에 `template: {componentId: 'item', dataBinding: '/items'}` 지정
- `root.properties.children.length === 2`
- 각 자식의 `text` 프로퍼티는 여전히 `{path: '.'}` (processor는 path를 해석하지 않고 컨텍스트만 설정)
- `processor.getData(child0, textProp0.path, 's1') === 'Item A'`
- `processor.getData(child1, textProp1.path, 's1') === 'Item B'`

### `it('getData resolves paths relative to node context')`
**검증 동작:** `getData`가 상대 경로, `.` 경로, 절대 경로를 모두 올바르게 해석하는지 확인한다.
- 픽스처: `user` → `{key: 'name', valueString: 'Alice'}` 설정
- `node = {id: 'test', dataContextPath: '/user'}`
- `getData(node, 'name', 's1') === 'Alice'` (상대 경로)
- `getData(node, '.', 's1')`이 `Map` 인스턴스이고 `.get('name') === 'Alice'` (`.` 경로)
- `getData(node, '/user/name', 's1') === 'Alice'` (절대 경로)

### `it('setData updates data model')`
**검증 동작:** `setData`로 루트 경로 및 중첩 경로에 값을 쓸 때 `dataModel`이 올바르게 갱신되는지 확인한다.
- `node = {id: 'test', dataContextPath: '/'}`
- `setData(node, 'count', 42)` → `dataModel.get('count') === 42`
- `setData(node, '/nested/value', 'foo')` → `dataModel.get('nested').get('value') === 'foo'`

### `it('normalizes paths correctly')`
**검증 동작:** 비공개 `normalizePath` 메서드가 브래킷 표기법 포함 경로를 슬래시 구분 경로로 정규화하는지 확인한다.
- `normalizePath('users[0].name') === '/users/0/name'`

### `it('parses JSON strings in data')`
**검증 동작:** `valueString`이 JSON 문자열처럼 보이는 경우(`{...}` 형태) 자동으로 파싱되어 객체로 저장되는지 확인한다.
- `{key: 'config', valueString: '{"theme":"dark"}'}` 전달
- `dataModel.get('config')` deepEqual `{theme: 'dark'}`

### `it('test basic edge cases and internal fallbacks')`
**검증 동작:** 여러 내부 경계 케이스와 폴백 동작을 하나의 테스트에서 순서대로 검증한다.
1. `clearSurfaces()` 호출 후 `surfaces.size === 0`
2. `setData(null, 'foo', 'bar')`가 예외 없이 종료됨 (null 노드 처리)
3. `dataContextPath`가 undefined인 노드에서 `setData(node, '.', 'value')`가 예외 없이 동작
4. 잘못된 JSON(`'{bad" }'`) 문자열은 파싱 실패 후 원본 문자열 그대로 저장됨
5. `setDataByPath`에 `unknownKey`만 있는 배열 요소를 전달하면 `Map`에 `'foo'` 키가 없음 (valueKey 없으면 스킵)
6. `setDataByPath(dataModel, '/', {plain: 'object'})`로 루트에 plain object를 설정하면 `dataModel.get('plain') === 'object'` (Object가 Map으로 정규화됨)
7. `setDataByPath(dataModel, '/', 'stringroot')`는 예외 없이 오류 로그만 출력

### `it('test array set operations and invalid primitive traversal')`
**검증 동작:** 배열 경로에 대한 `setDataByPath` 동작과 primitive 값을 통한 경로 탐색 실패를 검증한다.
- `dataModel`에 `'list'` 키로 `['dummy']` 배열 직접 설정
- `setDataByPath('/list/0/name', 'Alice')` → `list[0].get('name') === 'Alice'` (Map 자동 생성)
- `setDataByPath('/list/1', 'Bob')` → `list[1] === 'Bob'`
- `setDataByPath('/primitive', 'hello')` 후 `getDataByPath('/primitive/invalid') === null` (primitive 하위 탐색 불가)

### `it('test tree rebuilding edge cases')`
**검증 동작:** 트리 재구성의 경계 케이스를 검증한다.
- `surface.rootComponentId = null` 설정 후 `rebuildComponentTree` → `componentTree === null`
- 순환 참조(`circleA` → `circleB` → `circleA`) 상태에서 `rebuildComponentTree` 호출 시 `/Circular dependency/` 메시지 포함 오류가 throw됨

### `it('throws A2uiValidationError for malformed components')`
**검증 동작:** 각 컴포넌트 타입에서 필수 필드가 빠진 경우 `rebuildComponentTree`가 `/Invalid data; expected/` 메시지로 throw하는지 확인한다.
- 검증 대상 타입: `Text`, `Image`, `Icon`, `Video`, `AudioPlayer`, `Row`, `Column`, `List`, `Card`, `Tabs`, `Modal`, `Button`, `CheckBox`, `TextField`, `DateTimeInput`, `MultipleChoice`, `Slider` (총 17개, `Divider` 제외)
- 알 수 없는 타입(`CustomWidget`)은 예외 없이 통과하고 `type === 'CustomWidget'` 그대로 반환됨

### `it('resolves Template with Array data')`
**검증 동작:** `dataModelUpdate`에서 `path` 필드를 이용해 중첩 경로에 데이터를 쓰고, 템플릿이 배열 데이터를 기반으로 자식 노드를 생성하며 올바른 `id`와 `dataContextPath`를 부여하는지 확인한다.
- `dataModelUpdate({surfaceId, path: '/items', contents: [{key:'0', valueString:'a'},{key:'1',valueString:'b'}]})`
- 결과 `children.length === 2`
- `children[0].id === 'item:0'`, `children[0].dataContextPath === '/items/0'`
- `children[1].id === 'item:1'`, `children[1].dataContextPath === '/items/1'`

### `it('resolves Template with undefined/null data as empty array')`
**검증 동작:** `dataBinding`이 존재하지 않는 경로(`/missingItems`)를 가리킬 때 템플릿이 빈 배열로 확장되는지 확인한다.
- `children.length === 0`, `deepStrictEqual([], [])`

### `it('sets primitive at path via single array element with dot key')`
**검증 동작:** `contents: [{key: '.', valueString: 'hello'}]` 형태로 특정 경로에 primitive 값을 직접 설정하는 컨벤션이 올바르게 동작하는지 확인한다. 또한 `valueString`이 없는 `{key: '.'}` 요소를 가진 malformed 데이터는 `/must have exactly one value property/` 오류를 throw하는지 확인한다.
- `getData(root, '/nested/item', 's1_prim') === 'hello'`
- `{key: '.'}`만 있는 contents → `throws(/must have exactly one value property/)`

### `it('path resolves through primitive objects and arrays')`
**검증 동작:** `dataModel`에 plain object와 nested array가 혼합된 구조에서 `getData`가 중첩된 경로를 올바르게 탐색하는지 확인한다.
- `getData(root, '/obj/nestedMap/index/2/val', 's1_path') === 'hi'`
- `getData(root, '/obj/nestedMap/index/2/val/deeper', 's1_path') === null` (더 깊은 탐색 불가)

### `it('builds all component types correctly')`
**검증 동작:** 모든 지원 컴포넌트 타입(`Column`, `Row`, `Card`, `Tabs`, `Modal`, `List`, `Button`, `Text`, `Divider`, `CheckBox`, `TextField`, `DateTimeInput`, `MultipleChoice`, `Slider`, `Image`, `Icon`, `Video`, `AudioPlayer`)이 트리 구성 없이 각각 올바르게 빌드되는지 확인한다.
- 루트 `Column`에 10개의 다양한 타입 자식을 연결
- `root.type === 'Column'`, `root.properties.children.length === 10`

### `it('handles recursive valueMap in convertKeyValueArrayToMap')`
**검증 동작:** `valueMap` 내에 중첩된 `valueMap`이 재귀적으로 `Map`으로 변환되는지 확인한다.
- `dataModelUpdate`에 `{key: 'nested', valueMap: [{key: 'inner', valueString: 'val'}]}` 전달
- `dataModel.get('nested')` instanceof Map이고 `.get('inner') === 'val'`

### `it('resolves Template with Map data')`
**검증 동작:** 템플릿의 `dataBinding`이 `Map` 타입 데이터를 가리킬 때 Map의 각 엔트리에 대해 자식 노드가 생성되는지 확인한다.
- `items` Map에 `'a'`→`'valA'`, `'b'`→`'valB'` 두 항목
- 결과 `children.length === 2`

### `it('handles Object-to-Map normalization in handleDataModelUpdate (direct call)')`
**검증 동작:** `handleDataModelUpdate`에 `contents`가 배열 아닌 일반 객체로 전달될 때 key-value 쌍이 정상적으로 `dataModel`에 반영되는지 확인한다.
- 비공개 `handleDataModelUpdate` 직접 호출 (`(processor as any).handleDataModelUpdate(...)`)
- `dataModel.get('normalized') === 'value'`

### `it('throws validation error for invalid List')`
**검증 동작:** `List` 컴포넌트의 `children`이 배열이 아닌 문자열(`'not-array'`)일 때 `/Invalid data; expected List/` 오류가 throw되는지 확인한다.

### `it('handles recursive valueMap in convertKeyValueArrayToMap with dot key')`
**검증 동작:** `setDataByPath`에서 `contents`가 `[{key: '.', valueMap: [{key: 'inner', valueString: 'val'}]}]` 형태일 때 dot key에 의해 중첩 Map이 직접 경로에 저장되는지 확인한다.
- `target = surface.dataModel.get('target')`, instanceof Map이고 `.get('inner') === 'val'`

### `it('resolves Template with Array data (via direct setData)')`
**검증 동작:** `surface.dataModel`에 JavaScript 배열을 직접 설정했을 때 템플릿이 배열 길이에 맞는 자식 노드를 생성하는지 확인한다.
- `dataModel.set('items', ['A', 'B'])` 직접 설정
- `root.properties.children.length === 2`

### `it('ignores non-strings when trying to parse JSON')`
**검증 동작:** `parseIfJsonString`이 숫자 등 비문자열 값을 그대로 반환하는지 확인한다.
- `(processor as any).parseIfJsonString(123) === 123`

## 동작 흐름

`beforeEach`에서 `A2uiMessageProcessor` 인스턴스를 새로 생성하여 각 테스트가 독립적인 상태에서 시작하도록 보장한다. 각 테스트는 주로 `processMessages`를 통해 공개 API를 사용하며, 일부 테스트는 `(processor as any).비공개메서드(...)` 형태로 비공개 메서드를 직접 호출해 내부 로직을 검증한다. 모킹은 사용하지 않으며, 순수 인-메모리 상태 조작으로 검증한다.
