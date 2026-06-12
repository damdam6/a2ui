# renderers/react/tests/v0_9/catalog-components.test.tsx

## 개요

v0_9 기본 카탈로그(basic catalog)에 속하는 개별 React 컴포넌트들의 렌더링, 상호작용, 반응형 데이터 바인딩을 단위 테스트하는 파일이다. `renderA2uiComponent` 헬퍼를 통해 실제 A2UI 상태 라이프사이클을 유지하면서 각 컴포넌트를 격리하여 검증한다. Vitest와 @testing-library/react를 기반으로 하며, 모든 기본 카탈로그 컴포넌트(Text, Image, Icon, Video, AudioPlayer, Button, TextField, Row, Column, List, Card, Tabs, Modal, Divider, CheckBox, ChoicePicker, Slider, DateTimeInput)를 포함한다.

## 의존성

### 외부 패키지
- `vitest` — `describe`, `it`, `expect`, `vi`
- `@testing-library/react` — `screen`, `fireEvent`, `act`
- `@a2ui/web_core/v0_9` — `ComponentModel`

### 저장소 내부 모듈
- [`../utils`](../utils.tsx.md) — `renderA2uiComponent` 테스트 헬퍼
- [`../../src/v0_9/catalog/basic`](../../src/v0_9/catalog/basic/index.md) — `Text`, `Image`, `Icon`, `Video`, `AudioPlayer`, `Row`, `Column`, `List`, `Card`, `Tabs`, `Divider`, `Modal`, `Button`, `TextField`, `CheckBox`, `ChoicePicker`, `Slider`, `DateTimeInput`

## Exports

없음 (테스트 전용 파일, 직접 export 없음).

## 상세 명세 — 테스트 케이스

### `describe('Basic Catalog Components')`

#### `describe('Text')`

- **`renders static text`**: `Text` 컴포넌트를 `{text: 'Hello World'}` props로 렌더링하고, `screen.getByText('Hello World')`가 존재하는지 확인한다. `renderA2uiComponent(Text, 't1', {text: 'Hello World'})`를 사용한다.

- **`renders reactive text from data model`**: `{text: {path: '/msg'}}` 바인딩과 `{initialData: {msg: 'Initial'}}`로 렌더링한 후 초기 텍스트 'Initial'을 확인한다. 이후 `updateData('/msg', 'Updated')`를 `act`로 감싸 호출하고 텍스트가 'Updated'로 변경됨을 검증한다.

- **`renders with correct heading tag based on variant`**: `{text: 'Title', variant: 'h1'}` props로 렌더링하고 `view.container.querySelector('div.h1 h1')`가 null이 아니며 textContent가 `'Title'`임을 확인한다.

#### `describe('Image')`

- **`renders image with url and object-fit`**: `{url: 'https://example.com/img.png', fit: 'cover'}`로 렌더링하고 `img.src`와 `img.style.objectFit`이 각각 해당 값임을 확인한다.

- **`renders image with description as alt text`**: `{url: 'url', description: 'A beautiful sunset'}`으로 렌더링하고 `img.alt`가 `'A beautiful sunset'`임을 검증한다.

- **`applies variant-specific styling (avatar)`**: `{url: 'url', variant: 'avatar'}`로 렌더링하고 `img.style.borderRadius`가 `'50%'`이고 `img.style.width`가 `'var(--a2ui-image-avatar-size, 40px)'`임을 확인한다.

#### `describe('Icon')`

- **`renders material icon by name`**: `{name: 'settings'}`로 렌더링하고 컨테이너 textContent에 `'settings'`가 포함되고 `.material-symbols-outlined` 요소가 존재함을 확인한다.

- **`converts camelCase icon names to snake_case`**: `{name: 'shoppingCart'}`로 렌더링하고 textContent에 `'shopping_cart'`가 포함됨을 검증한다.

- **`maps "%s" to "%s"` (parameterized, `it.each`)**:  4가지 특수 아이콘 이름 매핑을 검증한다.
  - `'play'` → `'play_arrow'`
  - `'rewind'` → `'fast_rewind'`
  - `'favoriteOff'` → `'favorite_border'`
  - `'starOff'` → `'star_border'`
  각 경우 `{name: specName}`으로 렌더링하고 textContent에 materialName이 포함됨을 확인한다.

#### `describe('Video')`

- **`renders video element with source and controls`**: `{url: 'vid.mp4'}`로 렌더링하고 `video.src`에 `'vid.mp4'`가 포함되고 `video.controls`가 `true`임을 확인한다.

#### `describe('AudioPlayer')`

- **`renders audio element and description`**: `{url: 'audio.mp3', description: 'Listen to this'}`로 렌더링하고 `screen.getByText('Listen to this')`가 존재하며 `document.querySelector('audio').src`에 `'audio.mp3'`가 포함됨을 확인한다.

#### `describe('Button')`

- **`dispatches action on click`**: `{action: {event: {name: 'submit_clicked'}}, child: 'label1'}`로 렌더링 후 `surface.onAction`에 spy를 등록하고 `fireEvent.click(screen.getByRole('button'))`을 실행한다. spy가 `{name: 'submit_clicked'}`를 포함하는 객체와 함께 호출됐는지 검증한다.

- **`is disabled when isValid is false (via checks)`**: `checks` 배열에 `/name` 경로의 `required` 검사를 포함하는 버튼을 `{initialData: {name: ''}}` 로 렌더링하고 버튼이 disabled임을 확인한다. 이후 `updateData('/name', 'Alice')` 후 disabled가 `false`가 됨을 검증한다.

- **`delegates child rendering to buildChild`**: `{child: 'inner1'}`로 렌더링하고 `buildChild` mock이 `'inner1'`로 호출됐는지, `screen.getByTestId('child-inner1')`이 존재하는지 확인한다.

#### `describe('TextField')`

- **`updates data model on change`**: `{label: 'Name', value: {path: '/user/name'}}`으로 렌더링하고 `fireEvent.change`로 값을 `'Bob'`으로 변경한 뒤 `surface.dataModel.get('/user/name')`이 `'Bob'`임을 확인한다.

- **`shows validation error message`**: `checks`에 `required` 검사를 포함하는 TextField를 빈 email로 렌더링하고 `'Required!'` 메시지가 보임을 확인한다. 이후 `updateData('/email', 'test@test.com')` 후 메시지가 사라짐을 검증한다 (`queryByText`가 null).

#### `describe('Layout and Structural Components')`

- **`Row renders multiple children`**: `{children: ['c1', 'c2']}`로 Row 렌더링. `buildChild`가 `'c1'`, `'c2'` 각각으로 호출됐고 양쪽 testid 요소가 존재함을 확인한다.

- **`Column renders children vertically`**: `{children: ['c1']}`로 Column 렌더링. `buildChild`가 `'c1'`로 호출됐고 `view.container.firstChild`에 `{flexDirection: 'column'}` 스타일이 적용됐음을 확인한다.

- **`List supports dynamic templates with scoped data context`**: `{children: {componentId: 'itemComp', path: '/items'}}` 및 `{initialData: {items: [{n: 'A'}, {n: 'B'}]}, additionalImpls: [Text], additionalComponents: [new ComponentModel('itemComp', 'Text', {text: {path: 'n'}})]}`로 렌더링하고 `'A'`, `'B'` 텍스트가 화면에 나타남을 확인한다.

- **`Card renders its child`**: `{child: 'c1'}`로 Card 렌더링. `buildChild('c1')` 호출 및 `child-c1` testid 확인.

- **`Tabs switches active tab content`**: `{tabs: [{title: 'Home', child: 'home_c'}, {title: 'Settings', child: 'settings_c'}]}`로 렌더링한다. 초기에는 `child-home_c`가 보이고 `child-settings_c`는 없음을 확인한다. `fireEvent.click(screen.getByText('Settings'))` 후 반대로 바뀜을 검증한다.

- **`Modal opens content on trigger click`**: `{trigger: 't1', content: 'c1'}`로 렌더링. 초기에는 `child-t1`만 보이고 `child-c1`은 null. `fireEvent.click(screen.getByTestId('child-t1'))` 후 `child-c1`이 나타남을 확인한다.

- **`Divider renders a themed line`**: `{axis: 'horizontal'}`으로 Divider 렌더링. `firstChild`에 `{height: 'var(--a2ui-border-width, 1px)'}` 스타일이 적용됐음을 확인한다.

#### `describe('Input Components')`

- **`CheckBox updates data`**: `{label: 'Agree', value: {path: '/agreed'}}`로 렌더링. `fireEvent.click(screen.getByLabelText('Agree'))` 후 `surface.dataModel.get('/agreed')`가 `true`임을 확인한다.

- **`Slider updates data`**: `{label: 'Volume', value: {path: '/vol'}, max: 100}`으로 렌더링. `fireEvent.change`로 값을 `'75'`로 변경 후 `surface.dataModel.get('/vol')`이 숫자 `75`임을 확인한다 (문자열이 아닌 숫자로 변환됨).

- **`ChoicePicker mutuallyExclusive selection`**: `variant: 'mutuallyExclusive'`로 렌더링. 'A' 선택 후 `/picked`가 `['a']`, 'B' 선택 후 `['b']`로 배타적으로 변경됨을 확인한다.

- **`ChoicePicker filters options`**: `filterable: true`로 렌더링. 초기에 'Apple', 'Banana' 모두 보임. `fireEvent.change`로 filter 입력창에 `'App'` 입력 후 'Banana'가 사라짐을 확인한다. 입력창 placeholder는 `'Filter options...'`.

- **`ChoicePicker renders chips and handles selection`**: `displayStyle: 'chips'`로 렌더링. `fireEvent.click(screen.getByText('A'))` 후 `/picked`가 `['a']`임을 확인한다.

- **`DateTimeInput handles date changes`**: `{label: 'When', value: {path: '/date'}, enableDate: true}`로 렌더링. `fireEvent.change`로 값을 `'2026-03-20'`으로 변경 후 `/date`가 `'2026-03-20'`임을 확인한다.

## 동작 흐름

각 테스트는 `renderA2uiComponent`로 격리된 A2UI 환경을 생성하고 대상 컴포넌트를 실제 `SurfaceModel`/`ComponentModel` 인프라와 함께 렌더링한다. 반응형 데이터가 필요한 테스트는 `updateData` 헬퍼를 `act`로 감싸 상태 변경 후 재렌더링을 기다린다. 자식 컴포넌트가 필요한 경우 `buildChild` mock이 placeholder div를 반환하거나 (모델이 있으면) 실제 컴포넌트를 렌더링한다.
