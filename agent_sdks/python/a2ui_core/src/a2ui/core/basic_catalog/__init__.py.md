# agent_sdks/python/a2ui_core/src/a2ui/core/basic_catalog/__init__.py

## 개요

이 파일은 A2UI Basic Catalog 패키지의 공개 진입점이다. 카탈로그를 구성하는 컴포넌트 Pydantic 모델, 함수 API 클래스, 연산자 API 클래스, 테마 모델, 함수 구현체를 한 곳에서 re-export하고, 이를 조합하여 `BasicCatalog` 인스턴스를 제공한다. `BasicCatalog`는 `ModelCatalog`를 상속하며, 고정된 컴포넌트-이름 매핑과 함수 구현체를 생성자에서 설정한다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈

| import 출처 | 링크 |
|---|---|
| `.components` | [`./components.py`](./components.py.md) |
| `.function_apis` | [`./function_apis.py`](./function_apis.py.md) |
| `.operator_apis` | [`./operator_apis.py`](./operator_apis.py.md) |
| `.styles` | [`./styles.py`](./styles.py.md) |
| `.function_impls` | [`./function_impls.py`](./function_impls.py.md) |
| `..schema.constants` | [`../schema/constants.py`](../schema/constants.py.md) |
| `..catalog` | [`../catalog/__init__.py`](../catalog/__init__.py.md) |

## Exports

### 컴포넌트 클래스 (`.components`에서 re-export)
`AudioPlayerComponent`, `ButtonComponent`, `CardComponent`, `CheckBoxComponent`, `ChoicePickerComponent`, `ColumnComponent`, `DateTimeInputComponent`, `DividerComponent`, `IconComponent`, `ImageComponent`, `ListComponent`, `ModalComponent`, `RowComponent`, `SliderComponent`, `TabsComponent`, `TextComponent`, `TextFieldComponent`, `VideoComponent`, `AnyComponent`

### 함수 API 클래스 (`.function_apis`에서 re-export)
`RequiredApi`, `RegexApi`, `LengthApi`, `NumericApi`, `EmailApi`, `FormatStringApi`, `FormatNumberApi`, `FormatCurrencyApi`, `FormatDateApi`, `PluralizeApi`, `OpenUrlApi`, `AndApi`, `OrApi`, `NotApi`

### 연산자 API 클래스 (`.operator_apis`에서 re-export)
`AddApi`, `SubtractApi`, `MultiplyApi`, `DivideApi`, `EqualsApi`, `NotEqualsApi`, `GreaterThanApi`, `LessThanApi`, `ContainsApi`, `StartsWithApi`, `EndsWithApi`

### 기타
- `Theme` (`.styles`에서)
- `BASIC_FUNCTION_IMPLEMENTATIONS` (`.function_impls`에서)
- `BasicCatalog` (이 파일에서 정의)

---

## 상세 명세

### `_basic_catalog_id(spec_version: str) -> str`

모듈 내부 헬퍼 함수 (비공개, 이름 앞에 `_`).

`spec_version` 문자열을 받아 Basic Catalog의 JSON URL 식별자를 반환한다.

- `spec_version`의 `"."`을 `"_"`으로 치환한 버전 세그먼트를 사용.
- 반환 형식: `"{SPEC_BASE_URL}/{spec_version_underscored}/catalogs/basic/catalog.json"`
- 예: `spec_version = "v0.9"` → `"https://a2ui.org/.../v0_9/catalogs/basic/catalog.json"`

`SPEC_BASE_URL`은 `..schema.constants`에서 import된 상수다.

---

### `class BasicCatalog(ModelCatalog)`

Basic Catalog 전체를 하나의 객체로 묶는 구체 클래스. `ModelCatalog`를 상속한다.

#### `__init__(self) -> None`

인자 없음. 내부에서 18개 컴포넌트로 구성된 `components_map`을 하드코딩으로 정의한다.

`components_map`은 `Dict[str, Type[*Component]]` 형태로, 키는 컴포넌트 이름 문자열, 값은 해당 Pydantic 컴포넌트 클래스다:

| 키 | 값 |
|---|---|
| `"Text"` | `TextComponent` |
| `"Image"` | `ImageComponent` |
| `"Icon"` | `IconComponent` |
| `"Video"` | `VideoComponent` |
| `"AudioPlayer"` | `AudioPlayerComponent` |
| `"Row"` | `RowComponent` |
| `"Column"` | `ColumnComponent` |
| `"List"` | `ListComponent` |
| `"Card"` | `CardComponent` |
| `"Tabs"` | `TabsComponent` |
| `"Modal"` | `ModalComponent` |
| `"Divider"` | `DividerComponent` |
| `"Button"` | `ButtonComponent` |
| `"TextField"` | `TextFieldComponent` |
| `"CheckBox"` | `CheckBoxComponent` |
| `"ChoicePicker"` | `ChoicePickerComponent` |
| `"Slider"` | `SliderComponent` |
| `"DateTimeInput"` | `DateTimeInputComponent` |

이후 `super().__init__`을 다음 인자들로 호출한다:
- `spec_version=SPEC_VERSION` — `..schema.constants`에서 import된 현재 버전 문자열
- `catalog_id=_basic_catalog_id(SPEC_VERSION)` — 위 헬퍼로 생성한 카탈로그 JSON URL
- `components=components_map` — 위에서 정의한 18개 컴포넌트 매핑
- `functions=BASIC_FUNCTION_IMPLEMENTATIONS` — `.function_impls`에서 import된 구현체 딕셔너리
- `theme=Theme` — `.styles`에서 import된 Theme 클래스

추가적인 인스턴스 필드나 메서드를 정의하지 않으며, 모든 동작은 `ModelCatalog` 부모 클래스에 위임된다.

---

## 동작 흐름

패키지 import 시 모듈 수준에서 컴포넌트, 함수, 연산자, 스타일 모듈로부터 모든 심볼이 로드된다. `BasicCatalog()`를 인스턴스화하면 컴포넌트 이름-클래스 매핑, 함수 구현체, 테마 클래스, 버전 및 카탈로그 URL이 `ModelCatalog`로 전달되어 초기화된다. 외부 코드는 이 패키지를 import하는 것만으로 Basic Catalog의 전체 타입 시스템과 `BasicCatalog` 인스턴스에 접근할 수 있다.
