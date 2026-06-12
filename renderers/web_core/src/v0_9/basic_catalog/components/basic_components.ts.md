# renderers/web_core/src/v0_9/basic_catalog/components/basic_components.ts

## 개요

A2UI v0.9의 기본 컴포넌트 카탈로그를 정의하는 파일이다. 각 UI 컴포넌트의 이름(`name`)과 Zod 스키마(`schema`)로 구성된 `ComponentApi` 객체들을 선언하고, 이 모든 객체를 하나의 배열 `BASIC_COMPONENTS`로 집약하여 내보낸다. 렌더러 레이어가 허용된 컴포넌트 목록과 각 컴포넌트의 props 검증 규칙을 참조할 때 이 파일을 기준으로 삼는다.

## 의존성

### 외부 패키지
- `zod` — 런타임 스키마 검증 라이브러리. `z` 네임스페이스로 임포트.

### 저장소 내부 모듈
- [`../../schema/common-types.ts`](../../schema/common-types.ts.md) — `DynamicStringSchema`, `DynamicNumberSchema`, `DynamicBooleanSchema`, `DynamicStringListSchema`, `ChildListSchema`, `ComponentIdSchema`, `ActionSchema`, `AccessibilityAttributesSchema`, `CheckableSchema` 임포트.
- [`../../catalog/types.ts`](../../catalog/types.ts.md) — `ComponentApi` 타입 임포트.

## Exports

| 이름 | 종류 | 설명 |
|---|---|---|
| `TextApi` | `const` (`ComponentApi`) | Text 컴포넌트 API 정의 |
| `ImageApi` | `const` (`ComponentApi`) | Image 컴포넌트 API 정의 |
| `IconApi` | `const` (`ComponentApi`) | Icon 컴포넌트 API 정의 |
| `VideoApi` | `const` (`ComponentApi`) | Video 컴포넌트 API 정의 |
| `AudioPlayerApi` | `const` (`ComponentApi`) | AudioPlayer 컴포넌트 API 정의 |
| `RowApi` | `const` (`ComponentApi`) | Row 레이아웃 컴포넌트 API 정의 |
| `ColumnApi` | `const` (`ComponentApi`) | Column 레이아웃 컴포넌트 API 정의 |
| `ListApi` | `const` (`ComponentApi`) | List 컴포넌트 API 정의 |
| `CardApi` | `const` (`ComponentApi`) | Card 컴포넌트 API 정의 |
| `TabsApi` | `const` (`ComponentApi`) | Tabs 컴포넌트 API 정의 |
| `ModalApi` | `const` (`ComponentApi`) | Modal 컴포넌트 API 정의 |
| `DividerApi` | `const` (`ComponentApi`) | Divider 컴포넌트 API 정의 |
| `ButtonApi` | `const` (`ComponentApi`) | Button 컴포넌트 API 정의 |
| `TextFieldApi` | `const` (`ComponentApi`) | TextField 컴포넌트 API 정의 |
| `CheckBoxApi` | `const` (`ComponentApi`) | CheckBox 컴포넌트 API 정의 |
| `ChoicePickerApi` | `const` (`ComponentApi`) | ChoicePicker 컴포넌트 API 정의 |
| `SliderApi` | `const` (`ComponentApi`) | Slider 컴포넌트 API 정의 |
| `DateTimeInputApi` | `const` (`ComponentApi`) | DateTimeInput 컴포넌트 API 정의 |
| `BASIC_COMPONENTS` | `const` (`ComponentApi[]`) | 위 18개 API 객체를 선언 순서대로 담은 배열 |

## 상세 명세

### 모듈 수준 상수: `CommonProps`

비공개(`export` 없음) 상수. 모든 컴포넌트 스키마에 공통으로 포함되는 props 집합이다.

- `accessibility`: `AccessibilityAttributesSchema.optional()` — 접근성 속성. 선택 사항.
- `weight`: `z.number().optional()` — Row 또는 Column 내에서의 CSS `flex-grow`에 해당하는 상대적 가중치. Row/Column의 직계 자식일 때만 유효. 선택 사항.

---

### `TextApi`

- `name`: `'Text'`
- `schema`: strict object
  - `...CommonProps`
  - `text`: `DynamicStringSchema` (필수) — 표시할 텍스트. 기본 Markdown 지원(HTML·이미지·링크 제외).
  - `variant`: `'h1' | 'h2' | 'h3' | 'h4' | 'h5' | 'caption' | 'body'`, 기본값 `'body'`, 선택 사항. 기본 텍스트 스타일 힌트.

---

### `ImageApi`

- `name`: `'Image'`
- `schema`: strict object
  - `...CommonProps`
  - `url`: `DynamicStringSchema` (필수) — 이미지 URL.
  - `description`: `DynamicStringSchema.optional()` — 이미지 접근성 설명.
  - `fit`: `'contain' | 'cover' | 'fill' | 'none' | 'scaleDown'`, 기본값 `'fill'`, 선택 사항. CSS `object-fit`에 대응.
  - `variant`: `'icon' | 'avatar' | 'smallFeature' | 'mediumFeature' | 'largeFeature' | 'header'`, 기본값 `'mediumFeature'`, 선택 사항. 이미지 크기 및 스타일 힌트.

---

### `ICON_NAMES` (비공개 상수)

타입: `readonly string[]` (as const 튜플). 총 60개의 아이콘 이름 문자열을 열거한다. `IconApi`의 `name` 필드에서 `z.enum(ICON_NAMES)`로 사용된다. 전체 목록: `'accountCircle'`, `'add'`, `'arrowBack'`, `'arrowForward'`, `'attachFile'`, `'calendarToday'`, `'call'`, `'camera'`, `'check'`, `'close'`, `'delete'`, `'download'`, `'edit'`, `'event'`, `'error'`, `'fastForward'`, `'favorite'`, `'favoriteOff'`, `'folder'`, `'help'`, `'home'`, `'info'`, `'locationOn'`, `'lock'`, `'lockOpen'`, `'mail'`, `'menu'`, `'moreVert'`, `'moreHoriz'`, `'notificationsOff'`, `'notifications'`, `'pause'`, `'payment'`, `'person'`, `'phone'`, `'photo'`, `'play'`, `'print'`, `'refresh'`, `'rewind'`, `'search'`, `'send'`, `'settings'`, `'share'`, `'shoppingCart'`, `'skipNext'`, `'skipPrevious'`, `'star'`, `'starHalf'`, `'starOff'`, `'stop'`, `'upload'`, `'visibility'`, `'visibilityOff'`, `'volumeDown'`, `'volumeMute'`, `'volumeOff'`, `'volumeUp'`, `'warning'`.

---

### `IconApi`

- `name`: `'Icon'`
- `schema`: strict object
  - `...CommonProps`
  - `name`: union of three alternatives (필수):
    1. `z.enum(ICON_NAMES)` — 사전 정의된 아이콘 이름 중 하나.
    2. strict object `{ svgPath: z.string() }` — 커스텀 SVG path 데이터.
    3. strict object `{ path: z.string() }` — 파일 경로 기반 아이콘.

---

### `VideoApi`

- `name`: `'Video'`
- `schema`: strict object
  - `...CommonProps`
  - `url`: `DynamicStringSchema` (필수) — 비디오 URL.

---

### `AudioPlayerApi`

- `name`: `'AudioPlayer'`
- `schema`: strict object
  - `...CommonProps`
  - `url`: `DynamicStringSchema` (필수) — 오디오 URL.
  - `description`: `DynamicStringSchema.optional()` — 오디오 제목 또는 요약 설명.

---

### `RowApi`

- `name`: `'Row'`
- `schema`: strict object (최상위 `.describe(...)` 포함: "수평 배치 레이아웃 컴포넌트")
  - `...CommonProps`
  - `children`: `ChildListSchema` (필수) — 자식 목록. 고정 배열(문자열 ID) 또는 데이터 리스트 기반 템플릿 객체. 인라인 정의 불가, 반드시 ID 참조.
  - `justify`: `'center' | 'end' | 'spaceAround' | 'spaceBetween' | 'spaceEvenly' | 'start' | 'stretch'`, 기본값 `'start'`, 선택 사항. 주축(수평) 배치 방식.
  - `align`: `'start' | 'center' | 'end' | 'stretch'`, 기본값 `'stretch'`, 선택 사항. 교차축(수직) 정렬. CSS `align-items`에 대응.

---

### `ColumnApi`

- `name`: `'Column'`
- `schema`: strict object (최상위 `.describe(...)` 포함: "수직 배치 레이아웃 컴포넌트")
  - `...CommonProps`
  - `children`: `ChildListSchema` (필수) — RowApi와 동일한 규칙.
  - `justify`: `'start' | 'center' | 'end' | 'spaceBetween' | 'spaceAround' | 'spaceEvenly' | 'stretch'`, 기본값 `'start'`, 선택 사항. 주축(수직) 배치 방식.
  - `align`: `'center' | 'end' | 'start' | 'stretch'`, 기본값 `'stretch'`, 선택 사항. 교차축(수평) 정렬.

---

### `ListApi`

- `name`: `'List'`
- `schema`: strict object
  - `...CommonProps`
  - `children`: `ChildListSchema` (필수).
  - `direction`: `'vertical' | 'horizontal'`, 기본값 `'vertical'`, 선택 사항.
  - `align`: `'start' | 'center' | 'end' | 'stretch'`, 기본값 `'stretch'`, 선택 사항.
  - `listStyle`: `'ordered' | 'unordered' | 'none'`, 선택 사항 (기본값 없음).

---

### `CardApi`

- `name`: `'Card'`
- `schema`: strict object
  - `...CommonProps`
  - `child`: `ComponentIdSchema` (필수) — 카드 내부에 렌더링할 단일 자식 컴포넌트 ID. 복수 ID 전달 불가. 여러 요소 표시 시 Column/Row로 감싸야 한다.

---

### `TabsApi`

- `name`: `'Tabs'`
- `schema`: strict object
  - `...CommonProps`
  - `tabs`: 배열 (최소 1개), 각 요소는 strict object
    - `title`: `DynamicStringSchema` (필수) — 탭 제목.
    - `child`: `ComponentIdSchema` (필수) — 탭 콘텐츠 컴포넌트 ID.

---

### `ModalApi`

- `name`: `'Modal'`
- `schema`: strict object
  - `...CommonProps`
  - `trigger`: `ComponentIdSchema` (필수) — 모달을 여는 버튼 등 트리거 컴포넌트 ID.
  - `content`: `ComponentIdSchema` (필수) — 모달 내부에 표시할 컴포넌트 ID.

---

### `DividerApi`

- `name`: `'Divider'`
- `schema`: strict object
  - `...CommonProps`
  - `axis`: `'horizontal' | 'vertical'`, 기본값 `'horizontal'`, 선택 사항.

---

### `ButtonApi`

- `name`: `'Button'`
- `schema`: strict object
  - `...CommonProps`
  - `child`: `ComponentIdSchema` (필수) — 라벨용 Text 또는 아이콘 전용 버튼의 Icon ID.
  - `variant`: `'default' | 'primary' | 'borderless'`, 기본값 `'default'`, 선택 사항. `'primary'`는 주 CTA, `'borderless'`는 테두리/배경 없이 링크처럼 보이는 버튼.
  - `action`: `ActionSchema` (필수) — 클릭 시 실행할 액션.
  - `checks`: `CheckableSchema.shape.checks` — 체크 조건 배열.

---

### `TextFieldApi`

- `name`: `'TextField'`
- `schema`: strict object
  - `...CommonProps`
  - `label`: `DynamicStringSchema` (필수) — 입력 필드 레이블.
  - `value`: `DynamicStringSchema.optional()` — 텍스트 필드 현재 값.
  - `variant`: `'longText' | 'number' | 'shortText' | 'obscured'`, 기본값 `'shortText'`, 선택 사항.
  - `validationRegexp`: `z.string().optional()` — 클라이언트 측 입력 검증에 사용할 정규표현식.
  - `checks`: `CheckableSchema.shape.checks`.

---

### `CheckBoxApi`

- `name`: `'CheckBox'`
- `schema`: strict object
  - `...CommonProps`
  - `label`: `DynamicStringSchema` (필수) — 체크박스 옆 표시 텍스트.
  - `value`: `DynamicBooleanSchema` (필수) — 체크 상태 (`true`/`false`).
  - `checks`: `CheckableSchema.shape.checks`.

---

### `ChoicePickerApi`

- `name`: `'ChoicePicker'`
- `schema`: strict object (최상위 `.describe(...)` 포함: "하나 또는 여러 옵션을 선택하는 컴포넌트")
  - `...CommonProps`
  - `label`: `DynamicStringSchema.optional()` — 옵션 그룹 레이블.
  - `variant`: `'multipleSelection' | 'mutuallyExclusive'`, 기본값 `'mutuallyExclusive'`, 선택 사항.
  - `options`: 배열 (필수). 각 요소는 strict object:
    - `label`: `DynamicStringSchema` — 표시 텍스트.
    - `value`: `z.string()` — 해당 옵션의 안정적인 값.
  - `value`: `DynamicStringListSchema` (필수) — 현재 선택된 값 목록. 데이터 모델의 string 배열에 바인딩.
  - `displayStyle`: `'checkbox' | 'chips'`, 기본값 `'checkbox'`, 선택 사항.
  - `filterable`: `z.boolean()`, 기본값 `false`, 선택 사항. `true`이면 검색 입력란 표시.
  - `checks`: `CheckableSchema.shape.checks`.

---

### `SliderApi`

- `name`: `'Slider'`
- `schema`: strict object
  - `...CommonProps`
  - `label`: `DynamicStringSchema.optional()`.
  - `min`: `z.number()`, 기본값 `0`, 선택 사항.
  - `max`: `z.number()` (필수).
  - `step`: `z.number().optional()` — 슬라이더 단계 크기.
  - `value`: `DynamicNumberSchema` (필수) — 현재 값.
  - `checks`: `CheckableSchema.shape.checks`.

---

### `DateTimeInputApi`

- `name`: `'DateTimeInput'`
- `schema`: strict object
  - `...CommonProps`
  - `value`: `DynamicStringSchema` (필수) — ISO 8601 형식의 날짜/시간 값. 미설정 시 빈 문자열로 초기화.
  - `enableDate`: `z.boolean()`, 기본값 `false`, 선택 사항. `true`이면 날짜 선택 활성화.
  - `enableTime`: `z.boolean()`, 기본값 `false`, 선택 사항. `true`이면 시간 선택 활성화.
  - `min`: `z.union([DynamicStringSchema, z.string().date(), z.string().time(), z.string().datetime()]).optional()` — 최소 허용 날짜/시간 (ISO 8601).
  - `max`: 동일한 union 타입, 선택 사항. 최대 허용 날짜/시간.
  - `label`: `DynamicStringSchema.optional()`.
  - `checks`: `CheckableSchema.shape.checks`.

---

### `BASIC_COMPONENTS`

타입: `ComponentApi[]`. 위에서 정의된 18개의 API 객체를 선언 순서대로 담은 배열이다: `TextApi`, `ImageApi`, `IconApi`, `VideoApi`, `AudioPlayerApi`, `RowApi`, `ColumnApi`, `ListApi`, `CardApi`, `TabsApi`, `ModalApi`, `DividerApi`, `ButtonApi`, `TextFieldApi`, `CheckBoxApi`, `ChoicePickerApi`, `SliderApi`, `DateTimeInputApi`.

## 동작 흐름

이 파일은 실행 시 사이드 이펙트 없이 상수 선언만 수행한다. 모듈 로드 시 각 `ComponentApi` 객체가 순서대로 인스턴스화되고, 마지막에 `BASIC_COMPONENTS` 배열이 모든 객체를 참조하여 구성된다. 소비자 코드는 `BASIC_COMPONENTS`를 순회하거나 개별 `*Api` 객체를 임포트하여 컴포넌트 이름과 스키마를 활용한다. 각 스키마는 `satisfies ComponentApi`로 타입 호환성이 컴파일 타임에 검증된다.
