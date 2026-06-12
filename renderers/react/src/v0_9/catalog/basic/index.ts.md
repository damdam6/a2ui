# renderers/react/src/v0_9/catalog/basic/index.ts

## 개요

v0_9 basic catalog의 진입점(barrel) 파일이다. 모든 기본 컴포넌트 구현체를 배열로 수집하여 `Catalog` 인스턴스를 생성하고, 개별 컴포넌트와 `MarkdownContext` 관련 항목을 외부에 재수출한다. `basicCatalog`는 외부 렌더러가 a2ui 컴포넌트를 등록하기 위해 사용하는 핵심 객체다.

## 의존성

### 외부 패키지
- `@a2ui/web_core/v0_9` — `Catalog`
- `@a2ui/web_core/v0_9/basic_catalog` — `BASIC_FUNCTIONS`

### 저장소 내부 모듈
- [`../../adapter`](../../adapter.tsx.md) — `ReactComponentImplementation` (타입 import)
- [`./context/MarkdownContext`](./context/MarkdownContext.tsx.md) — `export *` 재수출
- [`./components/Text`](./components/Text.tsx.md) — `Text`
- [`./components/Video`](./components/Video.tsx.md) — `Video`
- [`./components/Row`](./components/Row.tsx.md) — `Row`
- [`./components/Tabs`](./components/Tabs.tsx.md) — `Tabs`
- [`./components/Modal`](./components/Modal.tsx.md) — `Modal`
- [`./components/TextField`](./components/TextField.tsx.md) — `TextField`
- [`./components/Slider`](./components/Slider.tsx.md) — `Slider`
- `./components/Image` — `Image`
- `./components/Icon` — `Icon`
- `./components/AudioPlayer` — `AudioPlayer`
- `./components/Column` — `Column`
- `./components/List` — `List`
- `./components/Card` — `Card`
- `./components/Divider` — `Divider`
- `./components/Button` — `Button`
- `./components/CheckBox` — `CheckBox`
- `./components/ChoicePicker` — `ChoicePicker`
- `./components/DateTimeInput` — `DateTimeInput`

## Exports

| 이름 | 종류 |
|------|------|
| `basicCatalog` | 상수 (`Catalog<ReactComponentImplementation>`) |
| `Text`, `Image`, `Icon`, `Video`, `AudioPlayer`, `Row`, `Column`, `List`, `Card`, `Tabs`, `Divider`, `Modal`, `Button`, `TextField`, `CheckBox`, `ChoicePicker`, `Slider`, `DateTimeInput` | 각 컴포넌트 구현체 상수 |
| `MarkdownContext`, `useMarkdownRenderer` | `export *` from `./context/MarkdownContext` |

## 상세 명세

### `basicComponents` (내부 상수)

`const basicComponents: ReactComponentImplementation[]`

총 18개의 컴포넌트 구현체를 포함하는 배열: `Text`, `Image`, `Icon`, `Video`, `AudioPlayer`, `Row`, `Column`, `List`, `Card`, `Tabs`, `Divider`, `Modal`, `Button`, `TextField`, `CheckBox`, `ChoicePicker`, `Slider`, `DateTimeInput`. 이 순서대로 `Catalog` 생성자에 전달된다.

### `basicCatalog`

`new Catalog<ReactComponentImplementation>(catalogUrl, basicComponents, BASIC_FUNCTIONS)`

- 첫 번째 인자: catalog을 식별하는 스펙 URL `'https://a2ui.org/specification/v0_9/catalogs/basic/catalog.json'`
- 두 번째 인자: `basicComponents` 배열
- 세 번째 인자: `BASIC_FUNCTIONS` — `@a2ui/web_core/v0_9/basic_catalog`에서 가져온 기본 함수 집합

## 동작 흐름

모듈이 로드될 때 모든 컴포넌트 모듈이 import되고, `basicComponents` 배열이 구성되며, `Catalog` 인스턴스가 단번에 생성된다. 이후 소비자는 `basicCatalog`를 a2ui 렌더러에 등록하거나, 개별 컴포넌트를 직접 import하여 사용할 수 있다.
