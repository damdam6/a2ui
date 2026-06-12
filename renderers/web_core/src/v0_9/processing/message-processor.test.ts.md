# renderers/web_core/src/v0_9/processing/message-processor.test.ts

## 개요

`MessageProcessor` 클래스의 동작을 검증하는 단위 테스트 파일이다. Node.js 내장 테스트 러너(`node:test`)와 `node:assert`를 사용하며, 메시지 처리, 능력(capabilities) 생성, 서피스 생명주기, 오류 처리 등 모든 주요 기능을 망라한다.

## 의존성

### 외부 패키지
- `node:assert` — 검증 유틸리티
- `node:test` — `describe`, `it`, `beforeEach`
- `zod` (`z`) — 테스트 픽스처용 스키마 정의

### 저장소 내부 모듈
- [`./message-processor.js`](./message-processor.ts.md) — 테스트 대상 클래스 `MessageProcessor`
- [`../catalog/types.js`](../catalog/types.ts.md) — `Catalog`, `ComponentApi`

## 픽스처 및 설정

`describe('MessageProcessor')` 블록 최상위에서 공유 변수를 선언한다:
- `processor: MessageProcessor<ComponentApi>` — 각 테스트에서 사용되는 프로세서 인스턴스
- `testCatalog: Catalog<ComponentApi>` — id가 `'test-catalog'`이고 컴포넌트가 없는 기본 카탈로그
- `actions: any[]` — 디스패치된 액션을 수집하는 배열

`beforeEach`에서 `actions = []`로 초기화하고, `new Catalog('test-catalog', [])`와 `new MessageProcessor<ComponentApi>([testCatalog], async a => { actions.push(a); })`로 매 테스트마다 새 인스턴스를 생성한다.

## 테스트 케이스

### `describe('getClientCapabilities')`

#### `generates basic client capabilities with supportedCatalogIds`
- 픽스처: 기본 `processor` (`test-catalog` 포함)
- 검증: `getClientCapabilities()`가 `{ 'v0.9': { supportedCatalogIds: ['test-catalog'] } }`를 반환하고 `inlineCatalogs`는 `undefined`임을 확인한다.

#### `generates inline catalogs when requested`
- 픽스처: `Button` 컴포넌트(`label: z.string()`)를 가진 `cat-1` 카탈로그
- 검증: `getClientCapabilities({ includeInlineCatalogs: true })` 결과의 첫 번째 `inlineCatalog`에서 `catalogId === 'cat-1'`, `Button` 스키마의 `allOf[0].$ref === 'common_types.json#/$defs/ComponentCommon'`, `allOf[1].properties.component.const === 'Button'`, `required === ['component', 'label']`임을 확인한다.

#### `transforms REF: descriptions into valid $ref nodes`
- 픽스처: `title: z.string().describe('REF:common_types.json#/$defs/DynamicString|The title')`를 가진 `Custom` 컴포넌트
- 검증: `title` 스키마에서 `$ref === 'common_types.json#/$defs/DynamicString'`, `description === 'The title'`, `type === undefined`임을 확인한다.

#### `generates inline catalogs with functions and theme schema`
- 픽스처: `Button` 컴포넌트, `add` 함수(파라미터 `a`, `b` 모두 숫자형), `primaryColor: z.string().describe('REF:...|The main color')` 테마 스키마를 가진 `cat-full` 카탈로그
- 검증: 함수 배열 길이 1, `fn.name === 'add'`, `fn.returnType === 'number'`, 파라미터 설명, 테마 `primaryColor.$ref === '...'`, `description === 'The main color'`을 확인한다.

#### `omits functions and theme when catalog has none`
- 픽스처: 컴포넌트만 있는 `cat-empty` 카탈로그
- 검증: `inlineCatalog.functions === undefined`, `inlineCatalog.theme === undefined`임을 확인한다.

#### `processes REF: tags deeply nested in schema arrays and objects`
- 픽스처: `items: z.array(z.object({ action: z.string().describe('REF:...Action|...') }))` 구조의 `DeepComp` 컴포넌트
- 검증: 중첩 배열 내부의 `action` 스키마에서 `$ref === '...Action'`, `type === undefined`임을 확인한다.

#### `handles REF: tags without pipes or with multiple pipes`
- 픽스처: `noPipe`(`REF:...NoPipe` — 파이프 없음), `multiPipe`(`REF:...MultiPipe|First|Second` — 파이프 두 개) 필드를 가진 `EdgeComp` 컴포넌트
- 검증: `noPipe.$ref === '...NoPipe'`, `noPipe.description === undefined`, `multiPipe.$ref === '...MultiPipe'`, `multiPipe.description === 'First'`(첫 번째 파이프 이후 값만 사용)임을 확인한다.

#### `handles multiple catalogs correctly`
- 픽스처: 컴포넌트만 있는 `cat-1`과, 함수 및 테마 스키마가 있는 `cat-2`로 구성된 `MessageProcessor`
- 검증: `inlineCatalogs` 길이 2, `cat-1`에 `functions === undefined` 및 `theme === undefined`, `cat-2`에 함수 1개와 테마가 존재함을 확인한다.

---

### 서피스 생명주기 테스트

#### `creates surface`
- 동작: `createSurface` 메시지(`surfaceId: 's1'`, `catalogId: 'test-catalog'`)를 처리한다.
- 검증: `processor.model.getSurface('s1')`이 존재하고 `surface.id === 's1'`, `surface.sendDataModel === false`임을 확인한다.

#### `creates surface with sendDataModel enabled`
- 동작: `sendDataModel: true`로 `createSurface` 메시지를 처리한다.
- 검증: `surface?.sendDataModel === true`임을 확인한다.

#### `getClientDataModel filters surfaces correctly`
- 동작: `s1`(sendDataModel: true, 데이터: `{user: 'Alice'}`), `s2`(sendDataModel: false, 데이터: `{secret: 'Bob'}`)를 생성 후 `getClientDataModel()`을 호출한다.
- 검증: 반환값 `version === 'v0.9'`, `surfaces.s1 === {user: 'Alice'}`, `surfaces.s2 === undefined`임을 확인한다.

#### `getClientDataModel returns undefined if no surfaces have sendDataModel enabled`
- 동작: `sendDataModel` 미지정으로 서피스를 생성한다.
- 검증: `getClientDataModel() === undefined`임을 확인한다.

#### `updates components on correct surface`
- 동작: `s1` 서피스에 `{id: 'root', component: 'Box'}` 컴포넌트를 추가한다.
- 검증: `surface?.componentsModel.get('root')`가 존재함을 확인한다.

#### `updates existing components via message`
- 동작: `btn` 컴포넌트를 `label: 'Initial'`로 생성한 뒤 동일 id로 `label: 'Updated'`를 전송한다.
- 검증: 첫 번째 처리 후 `btn.properties.label === 'Initial'`, 두 번째 처리 후 `btn.properties.label === 'Updated'`임을 확인한다.

#### `deletes surface`
- 동작: 서피스를 생성한 뒤 `deleteSurface` 메시지를 처리한다.
- 검증: 생성 후 서피스가 존재하고, 삭제 후 `getSurface('s1') === undefined`임을 확인한다.

#### `routes data model updates`
- 동작: 서피스 생성 후 `updateDataModel { path: '/foo', value: 'bar' }` 메시지를 처리한다.
- 검증: `surface?.dataModel.get('/foo') === 'bar'`임을 확인한다.

#### `notifies lifecycle listeners`
- 동작: `onSurfaceCreated`와 `onSurfaceDeleted` 구독 후 서피스 생성·삭제·재생성을 수행하고, 생성 구독 해제 후 또 다른 서피스를 생성한다.
- 검증: 생성 시 콜백이 올바른 서피스로 호출되고, 삭제 시 `deletedId === 's1'`이며, 구독 해제 후에는 콜백이 호출되지 않음을 확인한다.

---

### 오류 처리 테스트

#### `throws on message with multiple update types`
- 검증: 단일 메시지에 `updateComponents`와 `updateDataModel`을 동시에 포함하면 `/Message contains multiple update types/` 오류가 발생함을 확인한다.

#### `throws when creating component without type`
- 검증: `component` 필드 없이 컴포넌트를 생성하려 하면 `/Cannot create component comp1 without a type/` 오류가 발생하고 컴포넌트가 추가되지 않음을 확인한다.

#### `recreates component when type changes`
- 동작: `comp1`을 `Button` 타입으로 생성 후 `Label` 타입으로 다시 전송한다.
- 검증: 타입 변경 후 `comp.type === 'Label'`, `comp.properties.text === 'Lbl'`, `comp.properties.label === undefined`임을 확인한다.

#### `throws when catalog not found`
- 검증: 존재하지 않는 `catalogId: 'unknown-catalog'`로 서피스 생성 시 `/Catalog not found: unknown-catalog/` 오류가 발생함을 확인한다.

#### `throws when duplicate surface created`
- 검증: 동일 `surfaceId`로 두 번 `createSurface`를 호출하면 `/Surface s1 already exists/` 오류가 발생함을 확인한다.

#### `throws when updating non-existent surface`
- 검증: 존재하지 않는 `surfaceId: 'unknown-s'`로 `updateComponents`를 전송하면 `/Surface not found for message: unknown-s/` 오류가 발생함을 확인한다.

#### `throws when component is missing id`
- 검증: `id` 없이 `{ component: 'Button' }` 컴포넌트를 전송하면 `/missing an 'id'/` 오류가 발생함을 확인한다.

#### `throws when updating data on non-existent surface`
- 검증: 존재하지 않는 서피스에 `updateDataModel`을 전송하면 `/Surface not found for message: unknown-s/` 오류가 발생함을 확인한다.

---

### `describe('processMessages wrapper')`

#### `processes a list of messages`
- 동작: `{ messages: [...] }` 래퍼 형태로 `s1`, `s2` 두 서피스를 생성하는 메시지를 전달한다.
- 검증: 두 서피스 모두 모델에 존재함을 확인한다.

---

### `resolves paths correctly via resolvePath`
- 검증: 절대 경로는 그대로 반환, 상대 경로는 컨텍스트 경로 뒤에 붙여 반환, 컨텍스트 없으면 `'/' + path` 반환하는 동작을 4가지 입력/기대값 쌍으로 확인한다:
  - `('/foo', '/bar') → '/foo'`
  - `('foo', '/bar') → '/bar/foo'`
  - `('foo', '/bar/') → '/bar/foo'`
  - `('foo') → '/foo'`
