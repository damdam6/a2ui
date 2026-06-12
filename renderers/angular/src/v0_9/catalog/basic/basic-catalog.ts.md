# renderers/angular/src/v0_9/catalog/basic/basic-catalog.ts

## 개요

A2UI v0.9 basic catalog의 Angular 구현을 정의하는 파일이다. 18개 UI 컴포넌트(Text, Button, Row, Column 등)와 기본 함수 구현을 묶어 `AngularCatalog`로 패키징한다. `BasicCatalogBase` 클래스는 DI 없이 옵션으로 컴포넌트 오버라이드와 추가 컴포넌트를 지원하며, `BasicCatalog`는 Angular의 root-level `Injectable`로 제공되는 기본 구현체이다.

## 의존성

### 외부 패키지
- `@angular/core` — `Injectable`
- `@a2ui/web_core/v0_9/basic_catalog` — `BASIC_FUNCTIONS`, `TextApi`, `RowApi`, `ColumnApi`, `ButtonApi`, `TextFieldApi`, `ImageApi`, `IconApi`, `VideoApi`, `AudioPlayerApi`, `ListApi`, `CardApi`, `TabsApi`, `ModalApi`, `DividerApi`, `CheckBoxApi`, `ChoicePickerApi`, `SliderApi`, `DateTimeInputApi`
- `@a2ui/web_core/v0_9` — `FunctionImplementation`

### 저장소 내부 모듈
- [`../types`](../types.ts.md)
- [`./text.component`](./text.component.ts.md)
- [`./row.component`](./row.component.ts.md)
- [`./column.component`](./column.component.ts.md)
- [`./button.component`](./button.component.ts.md)
- [`./text-field.component`](./text-field.component.ts.md)
- [`./image.component`](./image.component.ts.md)
- [`./icon.component`](./icon.component.ts.md)
- [`./video.component`](./video.component.ts.md)
- [`./audio-player.component`](./audio-player.component.ts.md)
- [`./list.component`](./list.component.ts.md)
- [`./card.component`](./card.component.ts.md)
- [`./tabs.component`](./tabs.component.ts.md)
- [`./modal.component`](./modal.component.ts.md)
- [`./divider.component`](./divider.component.ts.md)
- [`./check-box.component`](./check-box.component.ts.md)
- [`./choice-picker.component`](./choice-picker.component.ts.md)
- [`./slider.component`](./slider.component.ts.md)
- [`./date-time-input.component`](./date-time-input.component.ts.md)

## Exports

- `BasicCatalogOptions` — 인터페이스
- `BASIC_COMPONENTS` — `AngularComponentImplementation[]` 상수
- `BASIC_FUNCTIONS` — 재익스포트 (외부 패키지에서 가져와 그대로 내보냄)
- `BasicCatalogBase` — 클래스
- `BasicCatalog` — Injectable 클래스

## 상세 명세

### `DEFAULT_COMPONENT_IMPLEMENTATIONS` (모듈 내부 상수)

타입: `Record<string, AngularComponentImplementation>`, `as const` 단언.

JSON 페이로드의 컴포넌트 타입 이름을 키로 사용하는 객체다. 프로퍼티 축소(property renaming)를 방지하기 위해 문자열 리터럴 키를 사용한다. 18개 항목이 있으며 각 항목은 `...XxxApi, component: XxxComponent` 형태로 구성된다:

| 키 | API | 컴포넌트 |
|---|---|---|
| `'text'` | `TextApi` | `TextComponent` |
| `'row'` | `RowApi` | `RowComponent` |
| `'column'` | `ColumnApi` | `ColumnComponent` |
| `'button'` | `ButtonApi` | `ButtonComponent` |
| `'textField'` | `TextFieldApi` | `TextFieldComponent` |
| `'image'` | `ImageApi` | `ImageComponent` |
| `'icon'` | `IconApi` | `IconComponent` |
| `'video'` | `VideoApi` | `VideoComponent` |
| `'audioPlayer'` | `AudioPlayerApi` | `AudioPlayerComponent` |
| `'list'` | `ListApi` | `ListComponent` |
| `'card'` | `CardApi` | `CardComponent` |
| `'tabs'` | `TabsApi` | `TabsComponent` |
| `'modal'` | `ModalApi` | `ModalComponent` |
| `'divider'` | `DividerApi` | `DividerComponent` |
| `'checkBox'` | `CheckBoxApi` | `CheckBoxComponent` |
| `'choicePicker'` | `ChoicePickerApi` | `ChoicePickerComponent` |
| `'slider'` | `SliderApi` | `SliderComponent` |
| `'dateTimeInput'` | `DateTimeInputApi` | `DateTimeInputComponent` |

### `BasicCatalogOptions` 인터페이스

카탈로그 구성을 위한 선택적 옵션 인터페이스다.

- `id?: string` — 카탈로그의 고유 식별자를 오버라이드한다. 기본값은 `'https://a2ui.org/specification/v0_9/catalogs/basic/catalog.json'`
- `components?: Partial<{[K in keyof typeof DEFAULT_COMPONENT_IMPLEMENTATIONS]: AngularComponentImplementation}>` — 특정 컴포넌트 구현을 교체할 때 사용한다
- `extraComponents?: AngularComponentImplementation[]` — 기본 18개 외 추가 컴포넌트를 등록한다
- `functions?: FunctionImplementation[]` — 기본 함수 구현(`BASIC_FUNCTIONS`)을 교체한다

### `BASIC_COMPONENTS` 상수

`Object.values(DEFAULT_COMPONENT_IMPLEMENTATIONS)`로 생성된 `AngularComponentImplementation[]` 배열이다. 기본 구현 객체의 모든 값을 순서대로 포함한다.

### `BasicCatalogBase` 클래스

`AngularCatalog`를 상속하는 클래스. DI 없이 직접 인스턴스화 가능하다.

**생성자** `(options: BasicCatalogOptions = {})`

1. `id`를 `options.id` 또는 기본 URI `'https://a2ui.org/specification/v0_9/catalogs/basic/catalog.json'`로 결정한다.
2. `functions`를 `options.functions` 또는 `BASIC_FUNCTIONS`로 결정한다.
3. `overrides`를 `options.components ?? {}`로 설정한다.
4. `DEFAULT_COMPONENT_IMPLEMENTATIONS`의 각 항목을 순회하여, 해당 키에 오버라이드가 있으면 오버라이드로, 없으면 기본값으로 사용한다. 최종 항목에는 `name`이 `impl.name || key`로 보장된다.
5. `options.extraComponents`가 있으면 배열에 추가한다.
6. `super(id, components, functions)`를 호출한다.

### `BasicCatalog` 클래스

`@Injectable({ providedIn: 'root' })` 데코레이터가 적용된 클래스로 `BasicCatalogBase`를 상속한다. 생성자는 `super()`를 인자 없이 호출하여 기본 옵션을 사용한다. Angular DI 컨테이너에 루트 스코프로 등록된다.

## 동작 흐름

모듈 로드 시 `DEFAULT_COMPONENT_IMPLEMENTATIONS` 상수가 초기화된다. `BasicCatalog`가 Angular DI에 의해 처음 요청되면 `BasicCatalogBase` 생성자가 실행되어 18개 컴포넌트와 기본 함수들이 `AngularCatalog` 인스턴스에 등록된다. 이후 렌더러가 JSON 페이로드의 컴포넌트 타입 이름으로 카탈로그를 조회하면 해당 Angular 컴포넌트가 반환된다.
