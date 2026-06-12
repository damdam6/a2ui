# renderers/angular/src/v0_8/catalog/index.ts

## 개요

v0.8 렌더러의 기본 컴포넌트 카탈로그를 정의하는 파일이다. a2ui 명세에서 지원하는 18개의 표준 UI 컴포넌트를 `Catalog` 타입의 객체로 등록한다. 또한 외부에서 임의의 `Catalog` 인스턴스에 표준 컴포넌트를 일괄 병합하는 헬퍼 함수도 제공한다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈
- [`../rendering/catalog`](../rendering/catalog.ts.md) — `Catalog` 타입
- [`../components/audio`](../components/audio.ts.md) — `AudioPlayer`
- [`../components/button`](../components/button.ts.md) — `Button`
- [`../components/card`](../components/card.ts.md) — `Card`
- [`../components/checkbox`](../components/checkbox.ts.md) — `Checkbox`
- [`../components/column`](../components/column.ts.md) — `Column`
- `../components/datetime-input` — `DateTimeInput`
- `../components/divider` — `Divider`
- `../components/icon` — `Icon`
- `../components/image` — `Image`
- `../components/list` — `List`
- `../components/modal` — `Modal`
- `../components/multiple-choice` — `MultipleChoice`
- `../components/row` — `Row`
- `../components/slider` — `Slider`
- `../components/tabs` — `Tabs`
- `../components/text` — `Text`
- `../components/text-field` — `TextField`
- `../components/video` — `Video`

## Exports

| 이름 | 종류 |
|------|------|
| `DEFAULT_CATALOG` | 상수 (`Catalog`) |
| `registerStandardComponents` | 함수 |

## 상세 명세

### 상수: `DEFAULT_CATALOG`
- 타입: `Catalog`
- 18개 키-값 쌍을 가진 객체 리터럴. 각 키는 a2ui 컴포넌트 타입 이름(문자열)이고, 값은 해당 Angular 컴포넌트 클래스를 동기적으로 반환하는 팩토리 함수 `() => ComponentClass` 형태다.
- 등록된 키와 대응 컴포넌트:
  - `'AudioPlayer'` → `AudioPlayer`
  - `'Button'` → `Button`
  - `'Card'` → `Card`
  - `'CheckBox'` → `Checkbox`
  - `'Column'` → `Column`
  - `'DateTimeInput'` → `DateTimeInput`
  - `'Divider'` → `Divider`
  - `'Icon'` → `Icon`
  - `'Image'` → `Image`
  - `'List'` → `List`
  - `'Modal'` → `Modal`
  - `'MultipleChoice'` → `MultipleChoice`
  - `'Row'` → `Row`
  - `'Slider'` → `Slider`
  - `'Tabs'` → `Tabs`
  - `'Text'` → `Text`
  - `'TextField'` → `TextField`
  - `'Video'` → `Video`

### 함수: `registerStandardComponents(catalog: Catalog): void`
- 매개변수: `catalog: Catalog` — 표준 컴포넌트를 추가로 등록할 대상 카탈로그 객체
- 반환 타입: `void`
- 동작: `Object.entries(DEFAULT_CATALOG)`를 순회하며, 각 `[key, value]` 쌍을 `catalog[key] = value`로 할당한다. 기존 카탈로그를 교체하지 않고 표준 컴포넌트를 병합하는 용도로 사용된다.

## 동작 흐름

모듈 로드 시 `DEFAULT_CATALOG` 객체가 즉시 생성된다. 각 컴포넌트는 팩토리 함수로 감싸져 있어 지연 해석이 가능하다. `registerStandardComponents`는 `DEFAULT_CATALOG`의 항목을 다른 카탈로그 인스턴스에 복사하는 단순 반복 작업을 수행한다.
