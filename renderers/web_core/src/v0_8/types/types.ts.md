# renderers/web_core/src/v0_8/types/types.ts

## 개요

A2UI 렌더러가 필요로 하는 모든 핵심 타입을 하나의 진입점으로 집약하는 중앙 타입 모듈이다. 서버-클라이언트 메시지 구조, 컴포넌트 트리, 데이터 모델, 테마(Theme), 서피스(Surface), 마크다운 렌더러 등 렌더러 구현에 필요한 완전한 타입 집합을 제공한다. 직접 정의하거나 내부 모듈에서 re-export하는 방식으로 구성되어 있으며 런타임 로직은 없다.

## 의존성

- 외부 패키지: `zod` (`z` 타입 임포트)
- 저장소 내부 모듈:
  - [`./client-event.ts`](./client-event.ts.md) — `ClientToServerMessage`(`A2UIClientEventMessage`로 re-export), `ClientCapabilitiesDynamic` re-export
  - [`./components.ts`](./components.ts.md) — `Action` re-export 및 내부 사용 (`AudioPlayer`, `Button`, `Card`, `Checkbox`, `Column`, `DateTimeInput`, `Divider`, `Icon`, `Image`, `List`, `Modal`, `MultipleChoice`, `Row`, `Slider`, `Tabs`, `Text`, `TextField`, `Video`)
  - [`./primitives.ts`](./primitives.ts.md) — `StringValue`, `NumberValue`, `BooleanValue` 임포트 및 re-export
  - `../schema/server-to-client.ts` — `A2uiMessageSchema`, `BeginRenderingMessageSchema`, `ComponentInstanceSchema`, `ComponentPropertiesSchema`, `DataModelUpdateMessageSchema`, `DeleteSurfaceMessageSchema`, `SurfaceUpdateMessageSchema`, `ValueMapSchema` 타입 임포트
  - `../schema/common-types.ts` — `ComponentArrayReferenceSchema`, `ComponentArrayTemplateSchema` 타입 임포트

## Exports

### Re-exports (다른 모듈에서 전달)

| 이름 | 원본 |
|------|------|
| `A2UIClientEventMessage` | `client-event.ts`의 `ClientToServerMessage` |
| `ClientCapabilitiesDynamic` | `client-event.ts` |
| `Action` | `components.ts` |
| `StringValue` | `primitives.ts` |
| `NumberValue` | `primitives.ts` |
| `BooleanValue` | `primitives.ts` |

### 이 파일에서 직접 정의하는 타입/인터페이스

| 이름 | 종류 |
|------|------|
| `MessageProcessor` | 타입 (객체 타입) |
| `Theme` | 타입 (객체 타입) |
| `UserAction` | 인터페이스 |
| `DataValue` | 타입 (재귀 유니언) |
| `DataObject` | 타입 |
| `DataMap` | 타입 |
| `DataArray` | 타입 |
| `ComponentArrayTemplate` | 인터페이스 |
| `ComponentArrayReference` | 인터페이스 |
| `ComponentProperties` | 인터페이스 |
| `ComponentInstance` | 인터페이스 |
| `BeginRenderingMessage` | 인터페이스 |
| `SurfaceUpdateMessage` | 인터페이스 |
| `DataModelUpdate` | 인터페이스 |
| `ValueMap` | 인터페이스 |
| `DeleteSurfaceMessage` | 인터페이스 |
| `ServerToClientMessage` | 인터페이스 |
| `ResolvedValue` | 타입 (재귀 유니언) |
| `ResolvedMap` | 타입 |
| `ResolvedArray` | 타입 |
| `BaseComponentNode` (비공개) | 인터페이스 |
| `TextNode` | 인터페이스 |
| `ImageNode` | 인터페이스 |
| `IconNode` | 인터페이스 |
| `VideoNode` | 인터페이스 |
| `AudioPlayerNode` | 인터페이스 |
| `RowNode` | 인터페이스 |
| `ColumnNode` | 인터페이스 |
| `ListNode` | 인터페이스 |
| `CardNode` | 인터페이스 |
| `TabsNode` | 인터페이스 |
| `DividerNode` | 인터페이스 |
| `ModalNode` | 인터페이스 |
| `ButtonNode` | 인터페이스 |
| `CheckboxNode` | 인터페이스 |
| `TextFieldNode` | 인터페이스 |
| `DateTimeInputNode` | 인터페이스 |
| `MultipleChoiceNode` | 인터페이스 |
| `SliderNode` | 인터페이스 |
| `CustomNode` | 인터페이스 |
| `AnyComponentNode` | 타입 (판별 유니언) |
| `ResolvedText` | 타입 별칭 |
| `ResolvedIcon` | 타입 별칭 |
| `ResolvedImage` | 타입 별칭 |
| `ResolvedVideo` | 타입 별칭 |
| `ResolvedAudioPlayer` | 타입 별칭 |
| `ResolvedDivider` | 타입 별칭 |
| `ResolvedCheckbox` | 타입 별칭 |
| `ResolvedTextField` | 타입 별칭 |
| `ResolvedDateTimeInput` | 타입 별칭 |
| `ResolvedMultipleChoice` | 타입 별칭 |
| `ResolvedSlider` | 타입 별칭 |
| `ResolvedRow` | 인터페이스 |
| `ResolvedColumn` | 인터페이스 |
| `ResolvedButton` | 인터페이스 |
| `ResolvedList` | 인터페이스 |
| `ResolvedCard` | 인터페이스 |
| `ResolvedTabItem` | 인터페이스 |
| `ResolvedTabs` | 인터페이스 |
| `ResolvedModal` | 인터페이스 |
| `CustomNodeProperties` | 인터페이스 |
| `SurfaceID` | 타입 별칭 |
| `Surface` | 인터페이스 |
| `MarkdownRenderer` | 타입 (함수 시그니처) |
| `MarkdownRendererTagClassMap` | 타입 |
| `MarkdownRendererOptions` | 타입 |

## 상세 명세

### `MessageProcessor`

렌더러가 서버-클라이언트 메시지를 처리하기 위해 구현해야 하는 인터페이스.

- `getSurfaces(): ReadonlyMap<string, Surface>` — 현재 모든 서피스를 읽기 전용 맵으로 반환
- `clearSurfaces(): void` — 모든 서피스 초기화
- `processMessages(messages: ServerToClientMessage[]): void` — 메시지 배열을 순차 처리
- `getData(node: AnyComponentNode, relativePath: string, surfaceId: string): DataValue | null` — 컴포넌트 노드의 데이터 컨텍스트 기준으로 상대 경로의 데이터를 반환. `.` 경로는 노드 자신의 데이터를 의미
- `setData(node: AnyComponentNode | null, relativePath: string, value: DataValue, surfaceId: string): void` — 지정 경로에 값을 저장
- `resolvePath(path: string, dataContextPath?: string): string` — 상대/절대 경로 문자열을 정규화

---

### `Theme`

렌더러 테마의 완전한 구조. 크게 세 섹션으로 나뉜다.

**`components`**: 각 컴포넌트별 Tailwind 클래스 맵. 단순 컴포넌트(`AudioPlayer`, `Button`, `Card`, `Column`, `Divider`, `Icon`, `List`, `Row`, `Video`)는 `Record<string, boolean>` 단일 맵을 갖는다. 복합 컴포넌트는 서브 구조를 가진다:
- `CheckBox`: `container`, `element`, `label`
- `DateTimeInput`: `container`, `element`, `label`
- `Image`: `all`, `icon`, `avatar`, `smallFeature`, `mediumFeature`, `largeFeature`, `header`
- `Modal`: `backdrop`, `element`
- `MultipleChoice`: `container`, `element`, `label`
- `Slider`: `container`, `element`, `label`
- `Tabs`: `container`, `element`, `controls.all`, `controls.selected`
- `Text`: `all`, `h1`, `h2`, `h3`, `h4`, `h5`, `caption`, `body`
- `TextField`: `container`, `element`, `label`

**`elements`**: `a`, `audio`, `body`, `button`, `h1`~`h5`, `iframe`, `input`, `p`, `pre`, `textarea`, `video` 등 HTML 기본 태그에 대한 클래스 맵 (`Record<string, boolean>`)

**`markdown`**: 마크다운 렌더링 시 태그별 CSS 클래스 배열. `p`, `h1`~`h5`, `ul`, `ol`, `li`, `a`, `strong`, `em` 을 포함 (`string[]`)

**`additionalStyles?`**: 각 컴포넌트에 적용할 인라인 스타일 오버라이드 맵 (`Record<string, string>`). `Text` 컴포넌트의 경우 플랫 맵 또는 `h1`~`h5`, `body`, `caption` 키를 가진 중첩 객체 중 하나를 허용하는 유니언 타입.

---

### `UserAction`

서버로 전송되는 사용자 동작 인터페이스 (이 파일 내 정의 버전).

- `actionName: string` — 컴포넌트의 `action.action` 속성에서 가져온 액션 이름
- `sourceComponentId: string` — 이벤트를 트리거한 컴포넌트의 `id`
- `timestamp: string` — ISO 8601 형식 타임스탬프
- `context?: { [k: string]: unknown }` — 데이터 바인딩이 해소된 컨텍스트 키-값 쌍

> 참고: `client-event.ts`의 `UserAction`과 유사하지만, 이 파일의 버전은 `surfaceId` 필드가 없고 `name` 대신 `actionName`을 사용한다.

---

### 데이터 모델 타입

- `DataValue`: `string | number | boolean | null | DataMap | DataObject | DataArray` — JSON-like 재귀 유니언 타입
- `DataObject`: `{ [key: string]: DataValue }` — 문자열 키 객체
- `DataMap`: `Map<string, DataValue>` — ES6 Map 기반 데이터 컨텍스트
- `DataArray`: `DataValue[]` — 값 배열

---

### `ComponentArrayTemplate`

데이터 모델의 목록으로부터 컴포넌트를 생성하는 템플릿.

- `extends z.infer<typeof ComponentArrayTemplateSchema>`
- 추가 필드:
  - `componentId: string` — 템플릿에 사용할 컴포넌트 ID
  - `dataBinding: string` — 데이터 소스 경로

---

### `ComponentArrayReference`

자식 컴포넌트 목록을 명시적 목록 또는 템플릿으로 표현하는 인터페이스.

- `extends z.infer<typeof ComponentArrayReferenceSchema>`
- 추가 필드:
  - `explicitList?: string[]` — 컴포넌트 ID 명시 목록
  - `template?: ComponentArrayTemplate` — 데이터 바인딩 기반 동적 생성 템플릿

---

### `ComponentProperties`

컴포넌트 인스턴스가 가질 수 있는 컴포넌트 속성들의 집합. `extends z.infer<typeof ComponentPropertiesSchema>`. 모든 컴포넌트 타입(`Text`, `Image`, `Icon`, `Video`, `AudioPlayer`, `Row`, `Column`, `List`, `Card`, `Tabs`, `Divider`, `Modal`, `Button`, `Checkbox`, `TextField`, `DateTimeInput`, `MultipleChoice`, `Slider`)을 선택적 필드로 포함한다.

---

### `ComponentInstance`

서버의 `SurfaceUpdate` 메시지에 포함된 원시 컴포넌트 인스턴스.

- `extends z.infer<typeof ComponentInstanceSchema>`
- `id: string`
- `weight?: number`
- `component: ComponentProperties`

---

### 메시지 인터페이스들

각 인터페이스는 대응하는 Zod 스키마를 `extends`하며 추가 필드를 선언한다.

| 인터페이스 | 핵심 필드 |
|---|---|
| `BeginRenderingMessage` | `surfaceId: string`, `root: string`, `styles?: { font?: string, primaryColor?: string }` |
| `SurfaceUpdateMessage` | `surfaceId: string`, `components: ComponentInstance[]` |
| `DataModelUpdate` | `surfaceId: string`, `path?: string`, `contents: ValueMap[]` |
| `ValueMap` | `key: string`, `valueString?: string`, `valueNumber?: number`, `valueBoolean?: boolean`, `valueMap?: ValueMap[]` — 자기 참조 재귀 타입 |
| `DeleteSurfaceMessage` | `surfaceId: string` |
| `ServerToClientMessage` | `beginRendering?: BeginRenderingMessage`, `surfaceUpdate?: SurfaceUpdateMessage`, `dataModelUpdate?: DataModelUpdate`, `deleteSurface?: DeleteSurfaceMessage` |

---

### 해소(Resolved) 타입 시스템

서버에서 받은 원시 컴포넌트 구조를 데이터 바인딩 해소 후 최종 렌더링 트리로 표현하는 타입 계층.

**`ResolvedValue`**: `string | number | boolean | null | AnyComponentNode | ResolvedMap | ResolvedArray` — 재귀 유니언

**`BaseComponentNode`** (비공개): 모든 노드 타입이 공유하는 기본 필드
- `id: string`
- `weight?: number`
- `dataContextPath?: string`
- `slotName?: string`

**컴포넌트별 Node 인터페이스**: 각각 `BaseComponentNode`를 상속하고 `type` 리터럴과 `properties` 타입을 갖는다.

| 인터페이스 | `type` 리터럴 | `properties` 타입 |
|---|---|---|
| `TextNode` | `'Text'` | `ResolvedText` |
| `ImageNode` | `'Image'` | `ResolvedImage` |
| `IconNode` | `'Icon'` | `ResolvedIcon` |
| `VideoNode` | `'Video'` | `ResolvedVideo` |
| `AudioPlayerNode` | `'AudioPlayer'` | `ResolvedAudioPlayer` |
| `RowNode` | `'Row'` | `ResolvedRow` |
| `ColumnNode` | `'Column'` | `ResolvedColumn` |
| `ListNode` | `'List'` | `ResolvedList` |
| `CardNode` | `'Card'` | `ResolvedCard` |
| `TabsNode` | `'Tabs'` | `ResolvedTabs` |
| `DividerNode` | `'Divider'` | `ResolvedDivider` |
| `ModalNode` | `'Modal'` | `ResolvedModal` |
| `ButtonNode` | `'Button'` | `ResolvedButton` |
| `CheckboxNode` | `'CheckBox'` | `ResolvedCheckbox` |
| `TextFieldNode` | `'TextField'` | `ResolvedTextField` |
| `DateTimeInputNode` | `'DateTimeInput'` | `ResolvedDateTimeInput` |
| `MultipleChoiceNode` | `'MultipleChoice'` | `ResolvedMultipleChoice` |
| `SliderNode` | `'Slider'` | `ResolvedSlider` |
| `CustomNode` | `string` (임의) | `CustomNodeProperties` |

**`AnyComponentNode`**: 위 19가지 Node 타입의 판별 유니언(discriminated union). `type` 필드로 구분한다.

**리프 컴포넌트의 Resolved 타입 별칭**: 자식을 포함하지 않아 원본 인터페이스를 그대로 재사용한다.
`ResolvedText = Text`, `ResolvedIcon = Icon`, `ResolvedImage = Image`, `ResolvedVideo = Video`, `ResolvedAudioPlayer = AudioPlayer`, `ResolvedDivider = Divider`, `ResolvedCheckbox = Checkbox`, `ResolvedTextField = TextField`, `ResolvedDateTimeInput = DateTimeInput`, `ResolvedMultipleChoice = MultipleChoice`, `ResolvedSlider = Slider`

**컨테이너 컴포넌트의 Resolved 인터페이스**:

| 인터페이스 | 핵심 필드 |
|---|---|
| `ResolvedRow` | `children: AnyComponentNode[]`, `distribution?: 'start'|'center'|'end'|'spaceBetween'|'spaceAround'|'spaceEvenly'`, `alignment?: 'start'|'center'|'end'|'stretch'` |
| `ResolvedColumn` | 동일 (`ResolvedRow`와 같은 구조) |
| `ResolvedButton` | `child: AnyComponentNode`, `action: Button['action']`, `primary?: boolean` |
| `ResolvedList` | `children: AnyComponentNode[]`, `direction?: 'vertical'|'horizontal'`, `alignment?: 'start'|'center'|'end'|'stretch'` |
| `ResolvedCard` | `child: AnyComponentNode`, `children: AnyComponentNode[]` |
| `ResolvedTabItem` | `title: StringValue`, `child: AnyComponentNode` |
| `ResolvedTabs` | `tabItems: ResolvedTabItem[]` |
| `ResolvedModal` | `entryPointChild: AnyComponentNode`, `contentChild: AnyComponentNode` |
| `CustomNodeProperties` | `{ [k: string]: ResolvedValue }` |

---

### `Surface`

하나의 UI 서피스 상태를 완전히 나타내는 인터페이스.

- `rootComponentId: string | null`
- `componentTree: AnyComponentNode | null`
- `dataModel: DataMap`
- `components: Map<string, ComponentInstance>`
- `styles: Record<string, string>`

---

### 마크다운 렌더러 타입

- `MarkdownRenderer`: `(markdown: string, options?: MarkdownRendererOptions) => Promise<string>` — 구현체는 XSS 방지를 위해 결과 HTML을 반드시 sanitize해야 한다.
- `MarkdownRendererTagClassMap`: `Record<string, string[]>` — 태그명 → 클래스명 배열 맵
- `MarkdownRendererOptions`: `{ tagClassMap?: MarkdownRendererTagClassMap }` — 렌더러에 전달되는 옵션

## 동작 흐름

이 파일은 타입 전용이다. 세 개의 내부 모듈(`client-event.ts`, `components.ts`, `primitives.ts`)과 두 개의 스키마 모듈(`server-to-client.ts`, `common-types.ts`)로부터 타입을 조합하여 렌더러 구현에 필요한 통합 API 표면을 제공한다. 특히 `AnyComponentNode` 판별 유니언과 `Resolved*` 타입 계층이 렌더러의 컴포넌트 트리 순회 로직의 타입 기반이 된다.
