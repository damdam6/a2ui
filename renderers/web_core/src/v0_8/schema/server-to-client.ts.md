# renderers/web_core/src/v0_8/schema/server-to-client.ts

## 개요

A2UI v0.8 서버→클라이언트 메시지 프로토콜 전체를 단일 최상위 Zod 스키마(`A2uiMessageSchema`)로 조립하는 파일이다. `common-types.ts`에서 개별 컴포넌트 스키마를 가져와 `AnyComponentSchema`, `ComponentInstanceSchema`, 그리고 네 가지 메시지 타입(BeginRendering, SurfaceUpdate, DataModelUpdate, DeleteSurface)을 구성한다. 특히 `SurfaceUpdateMessageSchema`는 컴포넌트 ID 중복과 존재하지 않는 ID 참조를 런타임에 검출하는 교차 검증 로직을 내장한다.

## 의존성

### 외부 패키지
- `zod`

### 저장소 내부 모듈
- [`./common-types.ts`](./common-types.ts.md) — `AudioPlayerSchema`, `ButtonSchema`, `CardSchema`, `CheckboxSchema`, `ColumnSchema`, `DateTimeInputSchema`, `DividerSchema`, `IconSchema`, `ImageSchema`, `ListSchema`, `ModalSchema`, `MultipleChoiceSchema`, `RowSchema`, `SliderSchema`, `TabsSchema`, `TextFieldSchema`, `TextSchema`, `VideoSchema`, `DataValueSchema`

## Exports

| 이름 | 종류 |
|------|------|
| `ValueMapSchema` | 상수 (Zod 스키마) |
| `AnyComponentSchema` | 상수 (Zod 스키마) |
| `ComponentPropertiesSchema` | 상수 (Zod 스키마, AnyComponentSchema의 별칭) |
| `ComponentInstanceSchema` | 상수 (Zod 스키마) |
| `BeginRenderingMessageSchema` | 상수 (Zod 스키마) |
| `SurfaceUpdateMessageSchema` | 상수 (Zod 스키마) |
| `DataModelUpdateMessageSchema` | 상수 (Zod 스키마) |
| `DeleteSurfaceMessageSchema` | 상수 (Zod 스키마) |
| `A2uiMessageSchema` | 상수 (Zod 스키마) |
| `A2uiMessage` | 타입 (infer) |

## 상세 명세

### 비공개 헬퍼: `_validateValueProperty`

시그니처: `(val: any, ctx: z.RefinementCtx) => void`

`valueString`, `valueNumber`, `valueBoolean`, `valueMap` 네 프로퍼티 중 정확히 하나만 정의되어 있는지 확인한다. 조건을 위반하면 커스텀 이슈를 추가한다. 파일 내에서 정의되지만 현재 코드에서는 직접 참조되지 않는다(`DataValueSchema`가 같은 역할을 수행).

---

### `ValueMapSchema`

`DataValueSchema`에 `.describe(...)` 설명을 추가한 스키마. 데이터 모델 항목 하나를 표현하며, `key`와 정확히 하나의 `value*` 프로퍼티를 함께 가져야 한다는 설명이 부착된다.

---

### `AnyComponentSchema`

지원되는 모든 컴포넌트 타입을 선택적 프로퍼티로 갖는 `z.object`. `.catchall(z.any())`로 알 수 없는 추가 키도 허용한다. 재귀 구조(`Row`, `Column`, `List`, `Card`)는 `z.lazy()`로 감싸진다.

지원 컴포넌트 키:
`Text`, `Image`, `Icon`, `Video`, `AudioPlayer`, `Row`, `Column`, `List`, `Card`, `Tabs`, `Divider`, `Modal`, `Button`, `Checkbox`, `TextField`, `DateTimeInput`, `MultipleChoice`, `Slider`

---

### `ComponentPropertiesSchema`

`AnyComponentSchema`의 직접 참조(별칭). 의미상 구분을 위해 별도 이름으로 export된다.

---

### `ComponentInstanceSchema`

UI 위젯 트리의 단일 컴포넌트 인스턴스를 표현하는 엄격(strict) 스키마.

- `id: string` — 컴포넌트 고유 식별자
- `weight?: number` — Row 또는 Column 직속 자식일 때만 사용 가능한 CSS `flex-grow`에 대응하는 상대적 가중치
- `component: ComponentPropertiesSchema` — 컴포넌트 타입 이름을 키로, 해당 프로퍼티 객체를 값으로 갖는 래퍼 객체. 정확히 하나의 키만 포함해야 한다.

---

### `BeginRenderingMessageSchema`

렌더링 시작 신호 메시지. 엄격 스키마.

- `surfaceId: string` — 렌더링할 UI 서피스 고유 식별자
- `catalogId?: string` — 컴포넌트 카탈로그 식별자. 생략 시 클라이언트는 A2UI v0.8 표준 카탈로그(`https://a2ui.org/specification/v0_8/standard_catalog_definition.json`)를 기본값으로 사용해야 한다.
- `root: string` — 렌더링할 루트 컴포넌트 ID
- `styles?`: 엄격 객체
  - `font?: string` — 기본 폰트
  - `primaryColor?: string` — `/^#[0-9a-fA-F]{6}$/` 패턴을 만족하는 6자리 16진수 색상 코드

---

### `SurfaceUpdateMessageSchema`

서피스의 컴포넌트 목록을 갱신하는 메시지. 엄격 스키마에 `superRefine`으로 교차 검증이 추가된다.

- `surfaceId: string` — 갱신 대상 서피스 식별자. 새 서피스인 경우 기존에 사용된 적 없는 새 ID여야 한다.
- `components: ComponentInstanceSchema[]` — 서피스의 모든 UI 컴포넌트 목록. 최소 1개 이상.

**교차 검증 로직 (`superRefine`)**:

1. **ID 중복 감지**: `components`를 순회하며 `Set<string>`에 각 `id`를 추가한다. 이미 존재하는 ID가 나타나면 `Duplicate component ID found: ${c.id}` 이슈를 추가한다.

2. **참조 무결성 검사 (`checkRefs`)**: 시그니처 `(ids: (string | undefined)[], componentId: string) => void`. 주어진 ID 배열 중 `undefined`가 아닌 값이 `componentIds` Set에 없으면 `Component '${componentId}' references non-existent component ID '${id}'.` 이슈를 추가한다.

3. **컴포넌트 타입별 참조 검사**: 각 컴포넌트를 순회하며 `component` 객체의 단일 키로 타입을 식별한 뒤 switch로 분기한다:
   - `Row` / `Column` / `List`: `children`이 `object`이면 `explicitList`와 `template` 중 정확히 하나가 있어야 함을 검증하고, 각각 `checkRefs`로 자식 ID 유효성을 확인한다.
   - `Card`: `child` ID를 `checkRefs`로 확인한다.
   - `Tabs`: 각 `tabItem`의 `child` ID를 `checkRefs`로 확인한다.
   - `Modal`: `entryPointChild`와 `contentChild` 두 ID를 `checkRefs`로 확인한다.
   - `Button`: `child` ID를 `checkRefs`로 확인한다.

---

### `DataModelUpdateMessageSchema`

서피스의 데이터 모델을 갱신하는 메시지. 엄격 스키마.

- `surfaceId: string` — 대상 서피스 식별자
- `path?: string` — 데이터 모델 내 갱신 위치 (예: `'/user/name'`). 생략하거나 `'/'`로 설정하면 전체 데이터 모델이 교체된다.
- `contents: ValueMapSchema[]` — 데이터 항목 배열. 각 항목은 `key`와 정확히 하나의 `value*` 프로퍼티를 포함해야 한다.

---

### `DeleteSurfaceMessageSchema`

서피스 삭제 신호 메시지. 엄격 스키마.

- `surfaceId: string` — 삭제할 서피스 식별자

---

### `A2uiMessageSchema`

A2UI 프로토콜 메시지의 최상위 래퍼 스키마. 엄격 객체에 `superRefine`으로 정확히 하나의 액션 프로퍼티 존재를 강제한다.

- `beginRendering?: BeginRenderingMessageSchema`
- `surfaceUpdate?: SurfaceUpdateMessageSchema`
- `dataModelUpdate?: DataModelUpdateMessageSchema`
- `deleteSurface?: DeleteSurfaceMessageSchema`

`superRefine`에서 네 키를 필터링하여 그 수가 1이 아니면 `A2UI Protocol message must have exactly one of: surfaceUpdate, dataModelUpdate, beginRendering, deleteSurface.` 이슈를 추가한다.

`A2uiMessage` 타입은 `z.infer<typeof A2uiMessageSchema>`로 도출되어 export된다.

## 동작 흐름

파일 로드 시: `_validateValueProperty` 헬퍼 정의 → `ValueMapSchema` 구성 → `AnyComponentSchema` 구성(재귀 컴포넌트는 `z.lazy`로 지연) → `ComponentPropertiesSchema` 별칭 → `ComponentInstanceSchema` 구성 → 네 가지 메시지 스키마 순서대로 구성 → `A2uiMessageSchema`가 이를 모두 감싸는 최상위 구조로 조립된다. 외부에서 `A2uiMessageSchema.parse()` 또는 `.safeParse()`를 호출하면 중첩된 스키마들이 안에서 바깥으로 순차 검증되고, `SurfaceUpdateMessageSchema`의 교차 검증이 마지막에 실행된다.
