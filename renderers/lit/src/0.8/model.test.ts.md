# renderers/lit/src/0.8/model.test.ts

## 개요

`v0_8.Data.A2uiMessageProcessor` 클래스의 단위 테스트 파일이다. 메시지 처리, 데이터 모델 갱신, 컴포넌트 트리 빌드, 템플릿 확장, 멀티 서피스 격리 등 프로세서의 핵심 동작을 검증한다. Node.js 내장 테스트 러너(`node:test`)와 어서션 모듈(`node:assert`)을 사용하며, `@ts-nocheck`로 TypeScript 타입 검사를 비활성화한다. 파일 상단에 `/* eslint-disable @typescript-eslint/ban-ts-comment */` 주석도 포함되어 있다.

## 의존성

### 외부 패키지
- `node:assert` — 표준 어서션 유틸리티
- `node:test` — `describe`, `it`, `beforeEach` 테스트 프레임워크 API
- `@a2ui/lit/v0_8` — `v0_8` 네임스페이스 전체 (`Data.A2uiMessageProcessor`, `Data.Guards`, `Types` 등 포함)
- `@a2ui/web_core/types/types` — `Types` 네임스페이스 (`AnyComponentNode`, `DataValue`, `DataMap` 등 타입)
- `@a2ui/web_core/v0_8` — `A2uiStateError` 에러 클래스

### 저장소 내부 모듈
패키지 경로(alias)로만 참조하여 직접적인 상대 경로 임포트는 없다.

## Exports

이 파일은 아무것도 export하지 않는다. 테스트 실행 엔트리포인트 전용이다.

## 상세 명세

### 헬퍼 함수

#### `toPlainObject(value: unknown): ReturnType<typeof JSON.parse>`

리액티브 래퍼(Lit Signal 등)를 제거하여 순수 JavaScript 객체로 변환하는 유틸리티다. 비교 어서션에서 리액티비티 구현 세부사항을 무시하기 위해 사용한다.

동작 단계:
1. `value`가 `Map` 인스턴스이면 `Array.from(value.entries())`로 항목을 추출한 뒤 각 값에 재귀 호출하여 일반 객체(`Object.fromEntries`)로 반환한다.
2. `value`가 배열이면 각 원소에 재귀 호출하여 새 배열로 반환한다.
3. `value`가 `v0_8.Data.Guards.isObject(value)`를 만족하고 `constructor.name === 'SignalObject'`이면, `Object.prototype.hasOwnProperty`로 own key만 순회하며 각 값에 재귀 호출한 후 평범한 객체로 반환한다.
4. 위 세 경우에 해당하지 않으면 `value`를 그대로 반환한다.

#### `assertIsDataMap(obj: Types.DataValue): asserts obj is Types.DataMap`

`obj`가 `Map` 인스턴스인지 단언하는 타입 가드 함수다. `assert.ok(obj instanceof Map, 'Data should be a DataMap')`로 검사하며, 실패 시 어서션 에러를 던진다. skip된 테스트에서 사용된다.

---

### 테스트 스위트: `A2uiMessageProcessor`

`beforeEach`에서 매 테스트 전 `processor = new v0_8.Data.A2uiMessageProcessor()`로 새 인스턴스를 생성한다.

---

#### `describe('Basic Initialization and State')`

**`it('should start with no surfaces')`**
- 검증 동작: 새로 생성된 프로세서의 `getSurfaces()` 맵 크기가 `0`임을 확인한다.
- 픽스처/모킹: 없음.

**`it('should clear surfaces when clearSurfaces is called')`**
- 검증 동작: `beginRendering` 메시지로 `'@default'` 서피스를 생성한 후 `clearSurfaces()`를 호출하면 `getSurfaces().size`가 다시 `0`이 됨을 확인한다.
- 픽스처: `{beginRendering: {root: 'root', surfaceId: '@default'}}`

---

#### `describe('Message Processing')`

**`it('should handle \`beginRendering\` by creating a default surface')`**
- 검증 동작: `beginRendering` 메시지에 `root: 'comp-a'`, `styles: {primaryColor: '#0000ff'}`, `surfaceId: '@default'`를 전달하면 서피스가 1개 생성되고, `rootComponentId`가 `'comp-a'`, `styles`가 `{primaryColor: '#0000ff'}`임을 확인한다.
- 픽스처: 인라인 메시지 배열.

**`it('should handle \`surfaceUpdate\` by adding components')`**
- 검증 동작: `beginRendering` 후 `surfaceUpdate`로 `Text` 컴포넌트를 추가하면 `surface.components.size === 1`이고 `rootComponentId`인 키가 포함됨을 확인한다.
- 픽스처: `surfaceId: '@default'`, `rootComponentId: 'comp-a'`, `Text` 컴포넌트.

**`it('should handle \`deleteSurface\`')`**
- 검증 동작: `beginRendering`으로 `'to-delete'` 서피스를 만든 뒤 `deleteSurface` 메시지를 보내면 해당 서피스가 `getSurfaces()`에 존재하지 않음을 확인한다.

---

#### `describe('Data Model Updates')`

**`it('should update data at a specified path')`**
- 검증 동작: `dataModelUpdate`로 `path: '/user'`, `contents: [{key: 'name', valueString: 'Alice'}]`를 전달한 뒤 `getData`로 `/user/name`을 조회하면 `'Alice'`가 반환됨을 확인한다.
- 픽스처: `surfaceId: '@default'`, `dataContextPath: '/'`.

**`it('should replace the entire data model when path is not provided')`**
- 검증 동작: `path: '/'`, `contents: [{key: 'user', valueString: JSON.stringify({name: 'Bob'})}]`를 전달하면 `/user` 조회 시 `{name: 'Bob'}`가 반환됨을 `toPlainObject`로 비교해 확인한다.

**`it('should create nested structures when setting data')`**
- 검증 동작: `processor.setData(component, '/a/b/c', 'value')` 호출 후 `getData(component, '/a/b/c')`가 `'value'`를 반환함을 확인한다. `setData`는 키-값 형식이 아닌 공개 메서드임을 주석으로 표기.

**`it('should handle paths correctly')`**
- 검증 동작: `resolvePath` 메서드의 세 가지 경로 조합을 확인한다.
  - `resolvePath('/a/b/c', '/value')` → `'/a/b/c'` (절대 경로는 그대로)
  - `resolvePath('a/b/c', '/value/')` → `'/value/a/b/c'` (상대 경로는 컨텍스트 아래로)
  - `resolvePath('a/b/c', '/value')` → `'/value/a/b/c'`

**`it.skip('should correctly parse nested valueMap structures')`** _(스킵됨)_
- 검증 동작: `valueMap` 중첩 구조가 `Map<string, Map<string, string>>` 형태로 파싱되어야 함을 검증하려 했으나 현재 비활성 상태.

**`it.skip('should additively update a Map using numeric-string keys (like timestamps)')`** _(스킵됨)_
- 검증 동작: 타임스탬프 형태의 숫자 문자열 키로 `Map`을 누적 갱신할 수 있어야 함을 검증하려 했으나 현재 비활성 상태. `assertIsDataMap` 헬퍼를 사용한다.

---

#### `describe('Component Tree Building')`

**`it('should build a simple parent-child tree')`**
- 검증 동작: `Column`(root) → `Text`(child) 구조의 컴포넌트를 등록하면 `componentTree`가 올바른 `id`, `type`, `properties.children` 구조를 가짐을 `toPlainObject`로 확인한다.
- 픽스처: `surfaceUpdate` 먼저, 이후 `beginRendering` 순서 (메시지 순서 역할을 검증).

**`it('should throw an error on circular dependencies')`**
- 검증 동작: `id: 'a'` → `child: 'b'`, `id: 'b'` → `child: 'a'` 순환 참조 컴포넌트를 등록한 후 `beginRendering`을 보내면 `A2uiStateError('Circular dependency for component "a".')`가 던져짐을 `assert.throws`로 확인한다. 또한 에러 후 `componentTree`가 `null`임을 검증한다.

**`it('should correctly expand a template with \`dataBinding\`')`**
- 검증 동작: `items: [{name: 'A'}, {name: 'B'}]` 데이터와 `dataBinding: '/items'` 템플릿 조합 시 `componentTree`의 `children`이 2개가 되고, 각 자식 노드의 `id`가 `'item-template:0'`/`'item-template:1'`, `dataContextPath`가 `'/items/0'`/`'/items/1'`, `properties.text`가 `{path: 'name'}`임을 확인한다.

**`it('should rebuild the tree when data for a template arrives later')`**
- 검증 동작: 먼저 `beginRendering`+`surfaceUpdate`를 보내 `children.length === 0`임을 확인한 뒤, 이후 `dataModelUpdate`로 `items` 데이터를 추가하면 `children.length === 2`로 갱신됨을 확인한다. 데이터 도착 후 트리가 자동 재빌드됨을 검증.

**`it('should trim relative paths within a data context (./item)')`**
- 검증 동작: 템플릿 내 `text: {path: './item/name'}` 경로가 데이터 컨텍스트 내에서 `{path: 'name'}`으로 정규화됨을 확인한다. (`/item`과 `./` 접두사 제거)

**`it('should trim relative paths within a data context (./name)')`**
- 검증 동작: 템플릿 내 `text: {path: './name'}` 경로가 데이터 컨텍스트 내에서 `{path: 'name'}`으로 정규화됨을 확인한다. (`./` 접두사 제거)

---

#### `describe('Data Normalization and Parsing')`

**`it('should correctly handle and parse the key-value array data format at the root')`**
- 검증 동작: `path: '/'`, `contents`에 `valueString: 'My Title'` 및 `valueString: '[{"id": 1}, {"id": 2}]'`를 전달하면 각각 문자열과 파싱된 배열로 조회됨을 확인한다. `surfaceId`로 `'test-surface'`를 명시적으로 지정하여 `getData`의 세 번째 인자로 사용.
- 픽스처: `surfaceId: 'test-surface'`.

**`it('should fallback to a string if stringified JSON is invalid')`**
- 검증 동작: 닫는 대괄호가 없는 잘못된 JSON 문자열 `'[{"id": 1}, {"id": 2}'`를 `valueString`으로 전달하면 파싱 실패 시 원본 문자열이 그대로 반환됨을 확인한다.

---

#### `describe('Complex Template Scenarios')`

**`it.skip('should correctly expand a template with dataBinding to a Map (from valueMap)')`** _(스킵됨)_
- 검증 동작: `valueMap` 형식으로 전달된 `items` Map을 `dataBinding`으로 참조할 때 5개의 자식 노드가 생성되고, 각 노드의 `id`가 `'item-card-template:item1'` 형태이며 `dataContextPath`가 `'/items/item1'`임을 깊은 트리 탐색으로 검증하려 했으나 현재 비활성.
- 픽스처: `Column` → `List` 구조의 복잡한 레스토랑 목록 UI 컴포넌트 트리 및 5개의 `valueMap` 항목.

**`it('should correctly expand nested templates with layered data contexts')`**
- 검증 동작: 외부 템플릿(`day-list`, `dataBinding: '/days'`)과 내부 중첩 템플릿(`activity-text`, `dataBinding: 'activities'`)을 조합할 때 계층적 데이터 컨텍스트가 올바르게 적용됨을 검증한다.
  - Day 1(`dataContextPath: '/days/0'`)의 activities가 2개이고 첫 번째 `id: 'activity-text:0:0'`, `dataContextPath: '/days/0/activities/0'`.
  - Day 2(`dataContextPath: '/days/1'`)의 activities가 1개이고 `id: 'activity-text:1:0'`, `dataContextPath: '/days/1/activities/0'`.
  - `path: 'title'`과 `path: '.'` 경로가 각 컨텍스트에서 유지됨.
- 픽스처: `days` 배열(`title`, `activities` 배열 포함).

**`it("should correctly bind to primitive values in an array using path: '.'")`**
- 검증 동작: 문자열 배열 `['travel', 'paris', 'guide']`에 `dataBinding: '/tags'`로 바인딩하고 `text: {path: '.'}` 경로를 사용하면 자식이 3개 생성되고 각 자식의 `dataContextPath`가 `'/tags/0'`, `'/tags/1'`이며 `properties.text === {path: '.'}` 임을 확인한다.

---

#### `describe('Multi-Surface Interaction')`

**`it('should keep data and components for different surfaces separate')`**
- 검증 동작: 두 개의 독립 서피스(`'A'`, `'B'`)에 각각 데이터와 컴포넌트를 설정하면 `getSurfaces().size === 2`이고 각 서피스의 `components`, `dataModel`, `componentTree`가 서로 독립적으로 유지됨을 확인한다.
  - Surface A: `comp-a`, `dataModel: {name: 'Surface A Data'}`, tree `text: {path: '/name'}`.
  - Surface B: `comp-b`, `dataModel: {name: 'Surface B Data'}`, tree `text: {path: '/name'}`.

## 동작 흐름

파일 로드 시 단일 `describe('A2uiMessageProcessor')` 블록이 등록된다. `beforeEach`가 각 테스트 전 프로세서를 초기화하므로 테스트 간 상태 오염이 없다. 각 테스트는 `processor.processMessages(messages)`로 메시지 배열을 전달하고, `getSurfaces()`, `getData()`, `setData()`, `resolvePath()` 등 공개 API를 통해 결과를 검증한다. `toPlainObject`는 리액티브 래퍼를 벗겨 순수 값 비교를 가능하게 한다. `it.skip`으로 표시된 4개 테스트는 현재 실행에서 제외된다.
