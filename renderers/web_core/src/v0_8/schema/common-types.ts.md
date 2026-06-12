# renderers/web_core/src/v0_8/schema/common-types.ts

## 개요

A2UI v0.8 프로토콜에서 사용하는 모든 공통 Zod 스키마와 TypeScript 타입을 정의한다. 서버→클라이언트 메시지에서 UI 컴포넌트 속성, 값 바인딩(데이터 경로 또는 리터럴), 사용자 액션 등을 검증하는 빌딩 블록 역할을 한다. 모든 스키마는 `zod`의 `.strict()` 및 `.superRefine()` 을 통해 불필요한 키와 일관성 없는 구조를 런타임에 거부한다.

## 의존성

### 외부 패키지
- `zod` — Zod 스키마 라이브러리 (`z`)

### 저장소 내부 모듈
없음

## Exports

| 이름 | 종류 |
|---|---|
| `StringValueSchema` | 상수 (Zod 스키마) |
| `StringValue` | TypeScript 타입 (infer) |
| `createDataValueSchema` | 함수 |
| `DataValueSchema` | 상수 (Zod 스키마) |
| `NumberValueSchema` | 상수 (Zod 스키마) |
| `NumberValue` | TypeScript 타입 (infer) |
| `BooleanValueSchema` | 상수 (Zod 스키마) |
| `BooleanValue` | TypeScript 타입 (infer) |
| `ActionSchema` | 상수 (Zod 스키마) |
| `TextSchema` | 상수 (Zod 스키마) |
| `ImageSchema` | 상수 (Zod 스키마) |
| `IconSchema` | 상수 (Zod 스키마) |
| `VideoSchema` | 상수 (Zod 스키마) |
| `AudioPlayerSchema` | 상수 (Zod 스키마) |
| `TabsSchema` | 상수 (Zod 스키마) |
| `DividerSchema` | 상수 (Zod 스키마) |
| `ModalSchema` | 상수 (Zod 스키마) |
| `ButtonSchema` | 상수 (Zod 스키마) |
| `CheckboxSchema` | 상수 (Zod 스키마) |
| `TextFieldSchema` | 상수 (Zod 스키마) |
| `DateTimeInputSchema` | 상수 (Zod 스키마) |
| `MultipleChoiceSchema` | 상수 (Zod 스키마) |
| `SliderSchema` | 상수 (Zod 스키마) |
| `ComponentArrayTemplateSchema` | 상수 (Zod 스키마) |
| `ComponentArrayReferenceSchema` | 상수 (Zod 스키마) |
| `RowSchema` | 상수 (Zod 스키마) |
| `ColumnSchema` | 상수 (Zod 스키마) |
| `ListSchema` | 상수 (Zod 스키마) |
| `CardSchema` | 상수 (Zod 스키마) |

## 상세 명세

### 비공개 헬퍼: `exactlyOneKey`

**시그니처**: `(val: any, ctx: z.RefinementCtx) => void`

`z.superRefine`에 전달되는 콜백. `Object.keys(val)`로 키 목록을 구한 뒤 `val[k] !== undefined`인 것만 필터링하여 수를 센다. 그 수가 정확히 1이 아니면 `z.ZodIssueCode.custom` 이슈를 ctx에 추가한다. 오류 메시지 형식: `"Must define exactly one property, found N (key1, key2, ...)."`. 여러 스키마에서 "값 종류 중 하나만 허용" 패턴을 재사용하기 위한 헬퍼다.

---

### `StringValueSchema`

**타입**: `z.ZodEffects<z.ZodObject<{ path?: string; literalString?: string; literal?: string }>>`

세 선택적 string 필드 중 정확히 하나만 정의되어야 한다. `.strict()`로 추가 키를 금지하고 `exactlyOneKey`로 유일성을 강제한다.

- `path`: 데이터 모델 경로 바인딩 (예: `'/user/name'`)
- `literalString`: 고정 문자열 값
- `literal`: 범용 리터럴 (문자열)

**`StringValue`**: `z.infer<typeof StringValueSchema>`로 도출한 TypeScript 타입.

---

### 비공개: `DataValueMapItemSchema`

**타입**: `z.ZodType<any>` — `z.lazy()`로 자기 자신을 재귀 참조

필드:
- `key: string` (필수)
- `valueString?: string`
- `valueNumber?: number`
- `valueBoolean?: boolean`
- `valueMap?: DataValueMapItemSchema[]`

`.strict()`으로 추가 키를 금지하고, `.superRefine()`에서 `valueString`, `valueNumber`, `valueBoolean`, `valueMap` 중 존재하는 것의 수를 카운트하여 정확히 1이 아니면 오류를 발생시킨다. 오류 메시지 형식: `"Value map item must have exactly one value property ..., found N."`. 재귀 구조 지원을 위해 `z.lazy()`로 감싸져 있다.

---

### `createDataValueSchema`

**시그니처**: `(options: { maxDepth?: number } = {}) => ZodEffects<...>`

`DataValueMapItemSchema`에 깊이 제한 검사를 추가한 새 스키마를 반환한다.

1. `options.maxDepth`를 읽고 없으면 `5`를 사용한다.
2. `DataValueMapItemSchema.superRefine()`에 내부 `checkDepth(v, currentDepth)` 함수를 등록한다.
3. `checkDepth`: `currentDepth > maxDepth`이면 `"valueMap recursion exceeded maximum depth of N."` 오류를 추가하고 반환. 그렇지 않으면 `v.valueMap`이 배열인 경우 각 항목에 `checkDepth(item, currentDepth + 1)`을 재귀 호출.
4. 루트 호출은 `checkDepth(val, 1)`로 시작한다.

---

### `DataValueSchema`

`createDataValueSchema()`를 기본 인수(maxDepth=5)로 호출한 결과 상수. `server-to-client.ts`의 `ValueMapSchema` 기반으로 사용된다.

---

### `NumberValueSchema`

**타입**: `z.ZodEffects<z.ZodObject<{ path?: string; literalNumber?: number; literal?: number }>>`

`StringValueSchema`와 동일한 구조이나 필드 타입이 `number`. `exactlyOneKey`로 유일성 강제. **`NumberValue`**: `z.infer<typeof NumberValueSchema>`.

---

### `BooleanValueSchema`

**타입**: `z.ZodEffects<z.ZodObject<{ path?: string; literalBoolean?: boolean; literal?: boolean }>>`

`StringValueSchema`와 동일한 구조이나 필드 타입이 `boolean`. `exactlyOneKey`로 유일성 강제. **`BooleanValue`**: `z.infer<typeof BooleanValueSchema>`.

---

### `ActionSchema`

**타입**: `z.ZodObject<{ name: string; context?: Array<...> }>`

- `name: string` — 액션 식별자 (예: `'submitForm'`)
- `context?: Array<{ key: string; value: { path?: string; literalString?: string; literalNumber?: number; literalBoolean?: boolean } }>` — 액션 발동 시 해석할 키-값 쌍 목록. `value` 객체는 `.strict()` + `exactlyOneKey`로 정확히 하나의 값 속성만 허용한다.

---

### `TextSchema`

`{ text: StringValueSchema; usageHint?: 'h1' | 'h2' | 'h3' | 'h4' | 'h5' | 'caption' | 'body' }`

---

### `ImageSchema`

`{ url: StringValueSchema; usageHint?: 'icon' | 'avatar' | 'smallFeature' | 'mediumFeature' | 'largeFeature' | 'header'; fit?: 'contain' | 'cover' | 'fill' | 'none' | 'scale-down'; altText?: StringValueSchema }`

---

### `IconSchema`

`{ name: StringValueSchema }` — 아이콘 이름을 StringValue로 바인딩.

---

### `VideoSchema`

`{ url: StringValueSchema }` — 비디오 URL.

---

### `AudioPlayerSchema`

`{ url: StringValueSchema; description?: StringValueSchema }` — 오디오 URL과 선택적 레이블.

---

### `TabsSchema`

`{ tabItems: Array<{ title: { path?: string; literalString?: string }; child: string }> }`

각 항목의 `.superRefine()`에서 세 가지를 순서대로 검사한다:
1. `title`이 falsy이면 `"Tab item is missing 'title'."` 오류 추가.
2. `child`가 falsy이면 `"Tab item is missing 'child'."` 오류 추가.
3. `title`이 있으면 `exactlyOneKey(val.title, ctx)` 호출로 title 내 유일성 추가 검사.

---

### `DividerSchema`

`{ axis?: 'horizontal' | 'vertical'; color?: string; thickness?: number }`

---

### `ModalSchema`

`{ entryPointChild: string; contentChild: string }` — 트리거 컴포넌트 ID와 콘텐츠 컴포넌트 ID.

---

### `ButtonSchema`

`{ child: string; action: ActionSchema; primary?: boolean }` — 버튼 콘텐츠 컴포넌트 ID, 액션, 주요 버튼 여부.

---

### `CheckboxSchema`

`{ label: StringValueSchema; value: { path?: string; literalBoolean?: boolean } }`

`value`는 `.strict()` + `exactlyOneKey`.

---

### `TextFieldSchema`

`{ text?: StringValueSchema; label: StringValueSchema; textFieldType?: 'shortText' | 'number' | 'date' | 'longText' | 'obscured'; validationRegexp?: string }`

`validationRegexp`는 입력값 검증에 사용할 정규식 문자열.

---

### `DateTimeInputSchema`

`{ value: StringValueSchema; enableDate?: boolean; enableTime?: boolean; outputFormat?: string }`

`outputFormat` 예시: `'YYYY-MM-DD'`.

---

### `MultipleChoiceSchema`

`{ selections: { path?: string; literalArray?: string[] }; options?: Array<{ label: ...; value: string }>; maxAllowedSelections?: number; type?: 'checkbox' | 'chips'; filterable?: boolean }`

- `selections`: `.strict()` + `exactlyOneKey`로 `path`/`literalArray` 중 하나
- `options[].label`: `{ path?: string; literalString?: string }` — `.strict()` + `exactlyOneKey`

---

### `SliderSchema`

`{ value: { path?: string; literalNumber?: number }; minValue?: number; maxValue?: number; label?: StringValueSchema }`

`value`는 `.strict()` + `exactlyOneKey`.

---

### `ComponentArrayTemplateSchema`

`{ componentId: string; dataBinding: string }` — 동적 목록 생성 템플릿. `componentId`는 템플릿 컴포넌트 ID, `dataBinding`은 데이터 모델 경로.

---

### `ComponentArrayReferenceSchema`

`{ explicitList?: string[]; template?: ComponentArrayTemplateSchema }` — `.strict()` + `exactlyOneKey`로 두 방식 중 하나만 허용.

- `explicitList`: 명시적 자식 컴포넌트 ID 배열
- `template`: 동적 생성 템플릿

---

### `RowSchema`

`{ children: ComponentArrayReferenceSchema; distribution?: 'start' | 'center' | 'end' | 'spaceBetween' | 'spaceAround' | 'spaceEvenly'; alignment?: 'start' | 'center' | 'end' | 'stretch' }`

---

### `ColumnSchema`

`RowSchema`와 동일한 구조. 수직 방향 레이아웃.

---

### `ListSchema`

`{ children: ComponentArrayReferenceSchema; direction?: 'vertical' | 'horizontal'; alignment?: 'start' | 'center' | 'end' | 'stretch' }`

---

### `CardSchema`

`{ child: string }` — 카드 내부에 렌더링할 컴포넌트 ID.

## 동작 흐름

파일 최상위에서 `exactlyOneKey` 헬퍼를 정의한 뒤, 이를 모든 값 타입 스키마의 `.superRefine()` 콜백으로 주입한다. `DataValueMapItemSchema`는 재귀 구조를 지원하기 위해 `z.lazy()`로 자기 참조하며, `createDataValueSchema()`가 그 위에 깊이 제한을 추가하고 `DataValueSchema` 상수로 즉시 인스턴스화된다. 컴포넌트 스키마들(TextSchema ~ CardSchema)은 독립적으로 선언된 단순 객체 스키마이며, `server-to-client.ts`에서 import하여 `AnyComponentSchema`를 구성하는 데 사용된다.
