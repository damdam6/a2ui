# renderers/web_core/src/v0_9/processing/message-processor.ts

## 개요

A2UI 서버-클라이언트 메시지 처리의 중앙 진입점이다. `MessageProcessor` 클래스는 카탈로그 목록을 인수로 받아 서피스 생성/삭제, 컴포넌트 업데이트, 데이터 모델 갱신 등 모든 메시지 유형을 라우팅한다. 또한 클라이언트 능력(capabilities) 객체를 인라인 카탈로그 JSON 스키마와 함께 생성하는 역할을 담당한다.

## 의존성

### 외부 패키지
- `zod-to-json-schema` — Zod 스키마를 JSON Schema로 변환

### 저장소 내부 모듈
- [`../state/surface-model.js`](../state/surface-model.ts.md) — `SurfaceModel`, `ActionListener`
- [`../catalog/types.js`](../catalog/types.ts.md) — `Catalog`, `ComponentApi`
- [`../state/surface-group-model.js`](../state/surface-group-model.ts.md) — `SurfaceGroupModel`
- [`../state/component-model.js`](../state/component-model.ts.md) — `ComponentModel`
- [`../common/events.js`](../common/events.ts.md) — `Subscription`
- [`../schema/server-to-client.js`](../schema/server-to-client.ts.md) — `A2uiMessage`, `CreateSurfaceMessage`, `UpdateComponentsMessage`, `UpdateDataModelMessage`, `DeleteSurfaceMessage`, `A2uiMessageListWrapper`
- [`../schema/client-capabilities.js`](../schema/client-capabilities.ts.md) — `A2uiClientCapabilities`, `InlineCatalog`
- [`../schema/client-to-server.js`](../schema/client-to-server.ts.md) — `A2uiClientDataModel`
- [`../errors.js`](../errors.ts.md) — `A2uiStateError`, `A2uiValidationError`

## Exports

| 이름 | 종류 |
|------|------|
| `CapabilitiesOptions` | 인터페이스 |
| `MessageProcessor` | 클래스 |

## 상세 명세

### 인터페이스 `CapabilitiesOptions`

`getClientCapabilities` 메서드에 전달하는 옵션 객체.

- `includeInlineCatalogs?: boolean` — `true`이면 응답에 모든 카탈로그의 전체 JSON Schema 정의를 포함한다.

---

### 클래스 `MessageProcessor<T extends ComponentApi>`

#### 필드

| 필드 | 타입 | 설명 |
|------|------|------|
| `model` | `SurfaceGroupModel<T>` (readonly) | 모든 서피스의 집합적 상태 모델 |
| `catalogs` | `Catalog<T>[]` (private) | 생성자에서 주입된 카탈로그 목록 |
| `actionHandler` | `ActionListener \| undefined` (private) | 모든 서피스 액션을 수신하는 전역 핸들러 |

#### 생성자 `constructor(catalogs: Catalog<T>[], actionHandler?: ActionListener)`

1. `this.model = new SurfaceGroupModel<T>()`로 그룹 모델을 초기화한다.
2. `actionHandler`가 제공된 경우 `this.model.onAction.subscribe(this.actionHandler)`로 구독을 등록한다.

---

#### `getClientCapabilities(options?: CapabilitiesOptions): A2uiClientCapabilities`

클라이언트가 서버에 전달해야 하는 능력(capabilities) 객체를 생성한다.

1. 기본 구조로 `{ 'v0.9': { supportedCatalogIds: [...카탈로그 id 배열] } }`를 생성한다.
2. `options?.includeInlineCatalogs`가 `true`이면 각 카탈로그에 대해 `generateInlineCatalog()`를 호출하여 `inlineCatalogs` 배열을 추가한다.
3. 완성된 `A2uiClientCapabilities` 객체를 반환한다.

---

#### `private generateInlineCatalog(catalog: Catalog<T>): InlineCatalog`

카탈로그 하나를 인라인 JSON Schema 표현으로 변환한다.

**컴포넌트 처리:**
1. `catalog.components`를 순회하며 각 컴포넌트의 Zod 스키마를 `zodToJsonSchema(..., { target: 'jsonSchema2019-09' })`로 변환한다.
2. `processRefs()`로 `REF:` 태그를 처리한다.
3. 결과를 A2UI 표준 envelope으로 감싼다:
   ```
   { allOf: [
     { $ref: 'common_types.json#/$defs/ComponentCommon' },
     { properties: { component: { const: name }, ...zodSchema.properties }, required: ['component', ...zodSchema.required] }
   ]}
   ```

**함수 처리:**
1. `catalog.functions`를 순회하며 각 함수의 스키마를 JSON Schema로 변환하고 `processRefs()`를 적용한다.
2. `{ name, description, returnType, parameters }` 객체를 `functions` 배열에 추가한다.
3. 함수가 하나도 없으면 `functions` 필드를 `undefined`로 설정한다.

**테마 처리:**
1. `catalog.themeSchema`가 존재하면 JSON Schema로 변환하고 `processRefs()`를 적용한 뒤 `.properties`만 추출해 `theme`에 할당한다.
2. 테마 스키마가 없으면 `theme`은 `undefined`이다.

반환값: `{ catalogId, components, functions?, theme? }`

---

#### `private processRefs(node: any): void`

JSON Schema 트리를 재귀적으로 탐색하여 `REF:` 접두사를 가진 `description` 필드를 `$ref` 노드로 변환한다.

1. `node`가 `null`이거나 `object`가 아니면 즉시 반환한다.
2. `node.description`이 문자열이고 `'REF:'`로 시작하면:
   - `'REF:'` 이후의 문자열을 `'|'`로 분리해 `parts`로 만든다.
   - `parts[0]`이 `$ref` 값, `parts[1]`(없으면 빈 문자열)이 설명 텍스트다.
   - `node`의 모든 기존 키를 삭제한다.
   - `node['$ref'] = ref`를 설정하고, 설명 텍스트가 있으면 `node['description'] = desc`를 설정한다.
   - 재귀를 중단하고 반환한다.
3. 배열이면 각 원소에 재귀 호출한다.
4. 그 외 객체이면 각 키의 값에 재귀 호출한다.

---

#### `getClientDataModel(): A2uiClientDataModel | undefined`

`sendDataModel`이 `true`인 서피스의 데이터 모델만 수집하여 반환한다.

1. `this.model.surfacesMap`을 순회하며 `surface.sendDataModel`이 `true`인 경우 `surfaces[surface.id] = surface.dataModel.get('/')`을 기록한다.
2. 수집된 서피스가 없으면 `undefined`를 반환한다.
3. 그렇지 않으면 `{ version: 'v0.9', surfaces }`를 반환한다.

---

#### `onSurfaceCreated(handler: (surface: SurfaceModel<T>) => void): Subscription`

`this.model.onSurfaceCreated.subscribe(handler)`를 호출하고 구독 객체를 반환한다.

---

#### `onSurfaceDeleted(handler: (id: string) => void): Subscription`

`this.model.onSurfaceDeleted.subscribe(handler)`를 호출하고 구독 객체를 반환한다.

---

#### `processMessages(messages: A2uiMessage[] | A2uiMessageListWrapper): void`

배열 또는 래퍼 객체 두 형태를 모두 처리한다.

1. `Array.isArray(messages)`이면 그대로, 아니면 `messages.messages`를 메시지 목록으로 사용한다.
2. 각 메시지에 대해 `processMessage()`를 순차적으로 호출한다.

---

#### `private processMessage(message: A2uiMessage): void`

단일 메시지를 적절한 처리 메서드로 라우팅한다.

1. `['createSurface', 'updateComponents', 'updateDataModel', 'deleteSurface']` 중 메시지에 존재하는 키를 필터링한다.
2. 두 개 이상 존재하면 `A2uiValidationError`를 던진다 — `"Message contains multiple update types: ..."`.
3. 키에 따라 `processCreateSurfaceMessage`, `processDeleteSurfaceMessage`, `processUpdateComponentsMessage`, `processUpdateDataModelMessage` 중 하나를 호출한다.

---

#### `private processCreateSurfaceMessage(message: CreateSurfaceMessage): void`

1. `payload.catalogId`로 카탈로그를 찾는다. 없으면 `A2uiStateError` `"Catalog not found: ..."`.
2. 동일 `surfaceId`의 서피스가 이미 존재하면 `A2uiStateError` `"Surface ... already exists."`.
3. `new SurfaceModel<T>(surfaceId, catalog, theme, sendDataModel ?? false)`를 생성하고 `this.model.addSurface()`에 추가한다.

---

#### `private processDeleteSurfaceMessage(message: DeleteSurfaceMessage): void`

`payload.surfaceId`가 존재하면 `this.model.deleteSurface(payload.surfaceId)`를 호출한다.

---

#### `private processUpdateComponentsMessage(message: UpdateComponentsMessage): void`

1. `payload.surfaceId`가 없으면 반환한다.
2. 해당 서피스가 없으면 `A2uiStateError` `"Surface not found for message: ..."`.
3. `payload.components`를 순회한다:
   - `id`가 없으면 `A2uiValidationError` `"Component '...' is missing an 'id'."`.
   - 기존 컴포넌트가 있고 타입이 변경된 경우: 기존 컴포넌트를 제거하고 새 `ComponentModel`을 생성·추가한다.
   - 기존 컴포넌트가 있고 타입이 같은 경우: `existing.properties = properties`로 속성만 갱신한다.
   - 컴포넌트가 없고 `component`(타입명)가 지정된 경우: `new ComponentModel(id, component, properties)`를 생성·추가한다.
   - 컴포넌트가 없고 타입도 없는 경우: `A2uiValidationError` `"Cannot create component ... without a type."`.

---

#### `private processUpdateDataModelMessage(message: UpdateDataModelMessage): void`

1. `payload.surfaceId`가 없으면 반환한다.
2. 해당 서피스가 없으면 `A2uiStateError` `"Surface not found for message: ..."`.
3. `path = payload.path || '/'`을 결정하고 `surface.dataModel.set(path, value)`를 호출한다.

---

#### `resolvePath(path: string, contextPath?: string): string`

경로를 절대 경로로 해석한다.

1. `path`가 `'/'`로 시작하면 그대로 반환한다.
2. `contextPath`가 제공된 경우 `contextPath` 끝에 `'/'`를 붙이고 `path`를 이어붙인다 (`contextPath`가 이미 `'/'`로 끝나는 경우 추가하지 않음).
3. `contextPath`가 없으면 `'/' + path`를 반환한다.

## 동작 흐름

외부에서 `processMessages()`를 호출하면 각 메시지가 `processMessage()`로 전달되어 메시지 유형 키에 따라 서피스 상태가 변경된다. 서피스가 생성될 때 `SurfaceGroupModel`은 `onSurfaceCreated` 이벤트를 발생시키고, 삭제 시에는 `onSurfaceDeleted`를 발생시킨다. 카탈로그 능력 노출은 `getClientCapabilities()`를 통해 이루어지며, Zod 스키마를 JSON Schema로 변환하고 `REF:` 태그를 재귀 처리하여 최종 `InlineCatalog` 객체를 구성한다.
