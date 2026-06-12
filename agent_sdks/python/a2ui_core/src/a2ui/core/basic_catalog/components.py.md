# agent_sdks/python/a2ui_core/src/a2ui/core/basic_catalog/components.py

## 개요

자동 생성된 파일로, a2ui basic catalog에서 사용하는 모든 UI 컴포넌트 Pydantic 모델을 정의한다. 각 컴포넌트 클래스는 `CatalogComponentCommon`을 상속하며, `component` 필드의 `Literal` 타입을 discriminator로 삼아 `AnyComponent` 유니온 타입으로 통합된다. 이 파일은 직접 수정하지 않는다.

## 의존성

### 외부 패키지
- `typing`: `Any`, `Dict`, `List`, `Literal`, `Optional`, `Union`, `Annotated`
- `pydantic`: `BaseModel`, `Field`, `ConfigDict`

### 저장소 내부 모듈
- [`../schema/common_types.py`](../schema/common_types.py.md): `StrictBaseModel`, `ComponentCommon`, `AccessibilityAttributes`, `DynamicString`, `DynamicNumber`, `DynamicBoolean`, `DynamicStringList`, `ChildList`, `Action`, `CheckRule`, `DataBinding`, `ComponentId`

## Exports

| 이름 | 종류 |
|---|---|
| `CatalogComponentCommon` | 클래스 (중간 기반 모델) |
| `OptionItem` | 클래스 (데이터 모델) |
| `SvgPath` | 클래스 (데이터 모델) |
| `TabItem` | 클래스 (데이터 모델) |
| `TextComponent` | 클래스 (컴포넌트 모델) |
| `ImageComponent` | 클래스 (컴포넌트 모델) |
| `IconComponent` | 클래스 (컴포넌트 모델) |
| `VideoComponent` | 클래스 (컴포넌트 모델) |
| `AudioPlayerComponent` | 클래스 (컴포넌트 모델) |
| `RowComponent` | 클래스 (컴포넌트 모델) |
| `ColumnComponent` | 클래스 (컴포넌트 모델) |
| `ListComponent` | 클래스 (컴포넌트 모델) |
| `CardComponent` | 클래스 (컴포넌트 모델) |
| `TabsComponent` | 클래스 (컴포넌트 모델) |
| `ModalComponent` | 클래스 (컴포넌트 모델) |
| `DividerComponent` | 클래스 (컴포넌트 모델) |
| `ButtonComponent` | 클래스 (컴포넌트 모델) |
| `TextFieldComponent` | 클래스 (컴포넌트 모델) |
| `CheckBoxComponent` | 클래스 (컴포넌트 모델) |
| `ChoicePickerComponent` | 클래스 (컴포넌트 모델) |
| `SliderComponent` | 클래스 (컴포넌트 모델) |
| `DateTimeInputComponent` | 클래스 (컴포넌트 모델) |
| `AnyComponent` | 타입 별칭 (Annotated Union) |

## 상세 명세

### `CatalogComponentCommon`
- 상속: `ComponentCommon`
- 필드:
  - `weight: Optional[float] = None` — Row 또는 Column의 직계 자식에서만 설정 가능하며 CSS `flex-grow`와 유사한 상대적 가중치.

---

### `OptionItem`
- 상속: `StrictBaseModel`
- 필드:
  - `label: DynamicString` — 옵션에 표시할 텍스트 (필수).
  - `value: str` — 이 옵션에 연결된 불변 값 (필수).

---

### `SvgPath`
- 상속: `StrictBaseModel`
- 필드:
  - `svg_path: str` — JSON 직렬화 시 alias `"svgPath"` 사용 (필수).

---

### `TabItem`
- 상속: `StrictBaseModel`
- 필드:
  - `title: DynamicString` — 탭 제목 (필수).
  - `child: ComponentId` — 탭 내용을 담는 자식 컴포넌트 ID; 인라인 정의 금지 (필수).

---

### `TextComponent`
- 상속: `CatalogComponentCommon`
- discriminator 값: `component: Literal["Text"] = "Text"`
- 필드:
  - `text: DynamicString` — 표시할 텍스트. 단순 Markdown 지원 (필수).
  - `variant: Optional[Literal["h1","h2","h3","h4","h5","caption","body"]] = "body"` — 기본 텍스트 스타일 힌트.

---

### `ImageComponent`
- 상속: `CatalogComponentCommon`
- discriminator 값: `component: Literal["Image"] = "Image"`
- 필드:
  - `url: DynamicString` — 이미지 URL (필수).
  - `description: Optional[DynamicString] = None` — 접근성 텍스트.
  - `fit: Optional[Literal["contain","cover","fill","none","scaleDown"]] = "fill"` — CSS `object-fit`에 해당.
  - `variant: Optional[Literal["icon","avatar","smallFeature","mediumFeature","largeFeature","header"]] = "mediumFeature"` — 이미지 크기/스타일 힌트.

---

### `IconComponent`
- 상속: `CatalogComponentCommon`
- discriminator 값: `component: Literal["Icon"] = "Icon"`
- 필드:
  - `name: Union[Literal[...], SvgPath, DataBinding]` — 표시할 아이콘 이름, SVG 경로, 또는 데이터 바인딩 (필수). 지원되는 Literal 이름 목록(56개): `"accountCircle"`, `"add"`, `"arrowBack"`, `"arrowForward"`, `"attachFile"`, `"calendarToday"`, `"call"`, `"camera"`, `"check"`, `"close"`, `"delete"`, `"download"`, `"edit"`, `"event"`, `"error"`, `"fastForward"`, `"favorite"`, `"favoriteOff"`, `"folder"`, `"help"`, `"home"`, `"info"`, `"locationOn"`, `"lock"`, `"lockOpen"`, `"mail"`, `"menu"`, `"moreVert"`, `"moreHoriz"`, `"notificationsOff"`, `"notifications"`, `"pause"`, `"payment"`, `"person"`, `"phone"`, `"photo"`, `"play"`, `"print"`, `"refresh"`, `"rewind"`, `"search"`, `"send"`, `"settings"`, `"share"`, `"shoppingCart"`, `"skipNext"`, `"skipPrevious"`, `"star"`, `"starHalf"`, `"starOff"`, `"stop"`, `"upload"`, `"visibility"`, `"visibilityOff"`, `"volumeDown"`, `"volumeMute"`, `"volumeOff"`, `"volumeUp"`, `"warning"`.

---

### `VideoComponent`
- 상속: `CatalogComponentCommon`
- discriminator 값: `component: Literal["Video"] = "Video"`
- 필드:
  - `url: DynamicString` — 영상 URL (필수).

---

### `AudioPlayerComponent`
- 상속: `CatalogComponentCommon`
- discriminator 값: `component: Literal["AudioPlayer"] = "AudioPlayer"`
- 필드:
  - `url: DynamicString` — 오디오 URL (필수).
  - `description: Optional[DynamicString] = None` — 오디오 제목 또는 요약 설명.

---

### `RowComponent`
- 상속: `CatalogComponentCommon`
- discriminator 값: `component: Literal["Row"] = "Row"`
- 필드:
  - `children: ChildList` — 고정 자식 ID 배열 또는 데이터 리스트 기반 템플릿 (필수).
  - `justify: Optional[Literal["center","end","spaceAround","spaceBetween","spaceEvenly","start","stretch"]] = "start"` — 주축(수평) 정렬.
  - `align: Optional[Literal["start","center","end","stretch"]] = "stretch"` — 교차축(수직) 정렬. CSS `align-items`에 해당.

---

### `ColumnComponent`
- 상속: `CatalogComponentCommon`
- discriminator 값: `component: Literal["Column"] = "Column"`
- 필드:
  - `children: ChildList` — 고정 자식 ID 배열 또는 데이터 리스트 기반 템플릿 (필수).
  - `justify: Optional[Literal["start","center","end","spaceBetween","spaceAround","spaceEvenly","stretch"]] = "start"` — 주축(수직) 정렬.
  - `align: Optional[Literal["center","end","start","stretch"]] = "stretch"` — 교차축(수평) 정렬.

---

### `ListComponent`
- 상속: `CatalogComponentCommon`
- discriminator 값: `component: Literal["List"] = "List"`
- 필드:
  - `children: ChildList` — 자식 항목 (필수).
  - `direction: Optional[Literal["vertical","horizontal"]] = "vertical"` — 항목 배치 방향.
  - `align: Optional[Literal["start","center","end","stretch"]] = "stretch"` — 교차축 정렬.

---

### `CardComponent`
- 상속: `CatalogComponentCommon`
- discriminator 값: `component: Literal["Card"] = "Card"`
- 필드:
  - `child: ComponentId` — 카드 내부에 렌더링할 단일 자식 ID; 여러 요소는 Column/Row로 감싸야 함 (필수).

---

### `TabsComponent`
- 상속: `CatalogComponentCommon`
- discriminator 값: `component: Literal["Tabs"] = "Tabs"`
- 필드:
  - `tabs: List[TabItem]` — 각각 `title`과 `child` ID를 갖는 탭 정의 배열 (필수).

---

### `ModalComponent`
- 상속: `CatalogComponentCommon`
- discriminator 값: `component: Literal["Modal"] = "Modal"`
- 필드:
  - `trigger: ComponentId` — 모달을 여는 트리거 컴포넌트 ID (필수).
  - `content: ComponentId` — 모달 내부에 표시할 컴포넌트 ID (필수).

---

### `DividerComponent`
- 상속: `CatalogComponentCommon`
- discriminator 값: `component: Literal["Divider"] = "Divider"`
- 필드:
  - `axis: Optional[Literal["horizontal","vertical"]] = "horizontal"` — 구분선 방향.

---

### `ButtonComponent`
- 상속: `CatalogComponentCommon`
- discriminator 값: `component: Literal["Button"] = "Button"`
- 필드:
  - `checks: Optional[List[CheckRule]] = None` — 버튼 활성화 전 실행할 검증 규칙 목록.
  - `child: ComponentId` — 버튼 레이블용 `Text` 또는 아이콘 전용일 때 `Icon` ID (필수).
  - `variant: Optional[Literal["default","primary","borderless"]] = "default"` — 버튼 스타일 힌트.
  - `action: Action` — 버튼 클릭 시 실행할 동작 (필수).

---

### `TextFieldComponent`
- 상속: `CatalogComponentCommon`
- discriminator 값: `component: Literal["TextField"] = "TextField"`
- 필드:
  - `checks: Optional[List[CheckRule]] = None` — 유효성 검사 규칙.
  - `label: DynamicString` — 입력 필드 레이블 (필수).
  - `value: Optional[DynamicString] = None` — 텍스트 필드 현재 값.
  - `variant: Optional[Literal["longText","number","shortText","obscured"]] = "shortText"` — 입력 필드 종류.
  - `validation_regexp: Optional[str] = None` — 클라이언트 측 유효성 검사용 정규식. alias `"validationRegexp"`.

---

### `CheckBoxComponent`
- 상속: `CatalogComponentCommon`
- discriminator 값: `component: Literal["CheckBox"] = "CheckBox"`
- 필드:
  - `checks: Optional[List[CheckRule]] = None` — 유효성 검사 규칙.
  - `label: DynamicString` — 체크박스 옆 텍스트 (필수).
  - `value: DynamicBoolean` — 체크 상태 (true=체크됨) (필수).

---

### `ChoicePickerComponent`
- 상속: `CatalogComponentCommon`
- discriminator 값: `component: Literal["ChoicePicker"] = "ChoicePicker"`
- 필드:
  - `checks: Optional[List[CheckRule]] = None` — 유효성 검사 규칙.
  - `label: Optional[DynamicString] = None` — 옵션 그룹 레이블.
  - `variant: Optional[Literal["multipleSelection","mutuallyExclusive"]] = "mutuallyExclusive"` — 다중/단일 선택 동작 힌트.
  - `options: List[OptionItem]` — 선택 가능한 옵션 목록 (필수).
  - `value: DynamicStringList` — 현재 선택된 값들의 배열 (필수).
  - `display_style: Optional[Literal["checkbox","chips"]] = "checkbox"` — 표시 스타일. alias `"displayStyle"`.
  - `filterable: Optional[bool] = False` — true이면 옵션 필터링 검색 입력란 표시.

---

### `SliderComponent`
- 상속: `CatalogComponentCommon`
- discriminator 값: `component: Literal["Slider"] = "Slider"`
- 필드:
  - `checks: Optional[List[CheckRule]] = None` — 유효성 검사 규칙.
  - `label: Optional[DynamicString] = None` — 슬라이더 레이블.
  - `min: Optional[float] = 0` — 최솟값.
  - `max: float` — 최댓값 (필수).
  - `value: DynamicNumber` — 현재 값 (필수).

---

### `DateTimeInputComponent`
- 상속: `CatalogComponentCommon`
- discriminator 값: `component: Literal["DateTimeInput"] = "DateTimeInput"`
- 필드:
  - `checks: Optional[List[CheckRule]] = None` — 유효성 검사 규칙.
  - `value: DynamicString` — ISO 8601 형식의 날짜/시간 값; 미설정 시 빈 문자열로 초기화 (필수).
  - `enable_date: Optional[bool] = False` — true이면 날짜 선택 활성화. alias `"enableDate"`.
  - `enable_time: Optional[bool] = False` — true이면 시간 선택 활성화. alias `"enableTime"`.
  - `min: Optional[DynamicString] = None` — ISO 8601 형식의 최솟값.
  - `max: Optional[DynamicString] = None` — ISO 8601 형식의 최댓값.
  - `label: Optional[DynamicString] = None` — 입력 필드 레이블.

---

### `AnyComponent`
- 타입: `Annotated[Union[TextComponent, ImageComponent, IconComponent, VideoComponent, AudioPlayerComponent, RowComponent, ColumnComponent, ListComponent, CardComponent, TabsComponent, ModalComponent, DividerComponent, ButtonComponent, TextFieldComponent, CheckBoxComponent, ChoicePickerComponent, SliderComponent, DateTimeInputComponent], Field(..., discriminator="component")]`
- 18개 컴포넌트 클래스의 판별 유니온 타입. `"component"` 필드의 리터럴 값을 discriminator로 사용하여 Pydantic이 역직렬화 시 올바른 클래스를 선택한다.

## 동작 흐름

이 파일은 순수 데이터 스키마 정의 파일이다. 모듈 로드 시 클래스 계층이 구성되고, `AnyComponent` 타입 별칭이 선언되어 외부에서 임포트 가능해진다. 런타임 로직은 없으며, Pydantic이 제공하는 유효성 검사·직렬화·역직렬화만 동작한다. `CatalogComponentCommon → ComponentCommon → StrictBaseModel`의 상속 계층을 통해 공통 컴포넌트 필드(접근성 속성, 스타일 등)가 모든 컴포넌트에 상속된다.
