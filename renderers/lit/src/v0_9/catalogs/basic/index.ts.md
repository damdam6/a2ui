# renderers/lit/src/v0_9/catalogs/basic/index.ts

## 개요

A2UI Lit 렌더러의 Basic Catalog를 조립하는 진입점 파일이다. 18개의 Lit 컴포넌트 구현체와 `BASIC_FUNCTIONS`를 하나의 `Catalog` 인스턴스로 묶어 `basicCatalog`로 내보낸다. Minimal Catalog보다 넓은 범위의 컴포넌트(리스트, 이미지, 아이콘, 비디오, 오디오 플레이어, 카드, 구분선, 체크박스, 슬라이더, 날짜/시간 입력, 선택 피커, 탭, 모달)를 포함한다.

## 의존성

### 외부 패키지
- `@a2ui/web_core/v0_9`: `Catalog`
- `@a2ui/web_core/v0_9/basic_catalog`: `BASIC_FUNCTIONS`
- `@a2ui/lit/v0_9`: `LitComponentApi`

### 저장소 내부 모듈
- [`./components/Text.js`](./components/Text.ts.md) — `A2uiText`
- [`./components/Button.js`](./components/Button.ts.md) — `A2uiButton`
- [`./components/TextField.js`](./components/TextField.ts.md) — `A2uiTextField`
- [`./components/Row.js`](./components/Row.ts.md) — `A2uiRow`
- [`./components/Column.js`](./components/Column.ts.md) — `A2uiColumn`
- [`./components/List.js`](./components/List.ts.md) — `A2uiList`
- [`./components/Image.js`](./components/Image.ts.md) — `A2uiImage`
- [`./components/Icon.js`](./components/Icon.ts.md) — `A2uiIcon`
- [`./components/Video.js`](./components/Video.ts.md) — `A2uiVideo`
- [`./components/AudioPlayer.js`](./components/AudioPlayer.ts.md) — `A2uiAudioPlayer`
- [`./components/Card.js`](./components/Card.ts.md) — `A2uiCard`
- [`./components/Divider.js`](./components/Divider.ts.md) — `A2uiDivider`
- [`./components/CheckBox.js`](./components/CheckBox.ts.md) — `A2uiCheckBox`
- [`./components/Slider.js`](./components/Slider.ts.md) — `A2uiSlider`
- [`./components/DateTimeInput.js`](./components/DateTimeInput.ts.md) — `A2uiDateTimeInput`
- [`./components/ChoicePicker.js`](./components/ChoicePicker.ts.md) — `A2uiChoicePicker`
- [`./components/Tabs.js`](./components/Tabs.ts.md) — `A2uiTabs`
- [`./components/Modal.js`](./components/Modal.ts.md) — `A2uiModal`

## Exports

- `basicCatalog` (상수): `Catalog<LitComponentApi>` 인스턴스 — Basic Catalog 전체

## 상세 명세

### `basicCatalog` 상수

- 타입: `Catalog<LitComponentApi>`
- `new Catalog<LitComponentApi>(id, components, functions)` 생성자로 생성된다.
  - `id`: `'https://a2ui.org/specification/v0_9/catalogs/basic/catalog.json'` — 카탈로그 식별 URL
  - `components`: 18개 컴포넌트 API 객체 배열 (`A2uiText`, `A2uiButton`, `A2uiTextField`, `A2uiRow`, `A2uiColumn`, `A2uiList`, `A2uiImage`, `A2uiIcon`, `A2uiVideo`, `A2uiAudioPlayer`, `A2uiCard`, `A2uiDivider`, `A2uiCheckBox`, `A2uiSlider`, `A2uiDateTimeInput`, `A2uiChoicePicker`, `A2uiTabs`, `A2uiModal`)
  - `functions`: `BASIC_FUNCTIONS` — `@a2ui/web_core`에서 제공하는 기본 함수 집합

## 동작 흐름

파일이 임포트될 때 모든 컴포넌트 모듈이 사이드이펙트로 로드되어 각 커스텀 엘리먼트(`@customElement(...)`)가 `customElements.define`을 통해 브라우저 레지스트리에 등록된다. 그 후 `basicCatalog` 인스턴스가 생성되어 `A2uiSurface` 또는 `MessageProcessor` 등에 전달된다. 컴포넌트 타입 문자열(예: `'Text'`, `'Button'`)에서 해당 Lit 엘리먼트의 `tagName`을 조회하는 매핑 역할을 한다.
