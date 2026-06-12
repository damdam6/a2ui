# renderers/react/tests/v0_8/integration/ThemeContext.test.tsx

## 개요

v0_8 컴포넌트들이 테마 클래스를 `ThemeContext`에서 올바르게 읽는지 검증하는 통합 테스트 파일이다. 컴포넌트가 `litTheme`을 직접 import하지 않고 `useTheme()`을 통해 테마를 소비하는지 확인하는 것이 핵심 목적이다. `TestWrapper`에 `theme` prop 없이 렌더링하면 기본 `litTheme` 클래스가 적용되고, 커스텀 모킹 테마를 전달하면 해당 클래스가 적용되는지를 각 컴포넌트별로 검증한다.

## 의존성

### 외부 패키지
- `vitest` — `describe`, `it`, `expect`
- `@testing-library/react` — `render`
- `react` — JSX
- `@a2ui/web_core/types/types` — `Types.Theme`, `Types.ServerToClientMessage` 타입 (타입 전용 import)

### 저장소 내부 모듈
- [`../utils`](../utils/index.ts.md) — `TestWrapper`, `TestRenderer`, `createSurfaceUpdate`, `createBeginRendering` (v0_8 테스트 유틸리티 re-export)
- [`../../../src/v0_8/theme/litTheme`](../../../src/v0_8/theme/litTheme.ts.md) — `litTheme` (기본 테마 객체, 검증 기준값으로 사용)

## Exports

테스트 파일이므로 외부로 export하는 항목 없음.

## 테스트 케이스 명세

### 헬퍼 함수: `createMockTheme(): Types.Theme`

비공개 팩토리 함수. 실제 `litTheme`과 완전히 다른 클래스명을 가진 모킹 테마 객체를 반환한다. 모든 컴포넌트 클래스에 `"test-"` 접두사를 붙여 목 테마 적용 여부를 DOM에서 명확히 식별할 수 있게 한다.

반환 객체 구조:
- `components` 맵: 아래 각 컴포넌트별 클래스 지정
  - `AudioPlayer`: `{'test-audioplayer': true}`
  - `Divider`: `{'test-divider': true}`
  - `Icon`: `{'test-icon': true}`
  - `Image`: `all: {'test-image-all': true}`, 나머지 서브슬롯(`avatar`, `header`, `icon`, `largeFeature`, `mediumFeature`, `smallFeature`)은 빈 객체
  - `Text`: `all: {'test-text-all': true}`, `h1: {'test-text-h1': true}`, `h2: {'test-text-h2': true}`, 나머지 서브슬롯 빈 객체
  - `Video`: `{'test-video': true}`
  - `Card`: `{'test-card': true}`
  - `Column`: `{'test-column': true}`
  - `List`: `{'test-list': true}`
  - `Modal`: `backdrop: {}`, `element: {'test-modal-element': true}`
  - `Row`: `{'test-row': true}`
  - `Tabs`: `container: {'test-tabs-container': true}`, `controls: {all: {'test-tabs-control': true}, selected: {'test-tabs-selected': true}}`, `element: {'test-tabs-element': true}`
  - `Button`: `{'test-button': true}`
  - `CheckBox`: `container: {'test-checkbox-container': true}`, `element: {}`, `label: {}`
  - `DateTimeInput`: `container: {'test-datetime-container': true}`, `label: {}`, `element: {}`
  - `MultipleChoice`: `container: {'test-multiplechoice-container': true}`, `label: {}`, `element: {}`
  - `Slider`: `container: {'test-slider-container': true}`, `label: {}`, `element: {}`
  - `TextField`: `container: {'test-textfield-container': true}`, `label: {}`, `element: {}`
- `elements` 맵: `a`, `audio`, `body`, `button`, `h1`~`h5`, `iframe`, `input`, `p`, `pre`, `textarea`, `video` 모두 빈 객체
- `markdown` 맵: `p`, `h1`~`h5`, `ul`, `ol`, `li`, `a`, `strong`, `em` 모두 빈 배열

---

### `describe('ThemeContext') > describe('Theme Switching')`

모든 케이스에서 공통 패턴: `createSurfaceUpdate([...컴포넌트 목록])` + `createBeginRendering(루트ID)` 메시지 배열을 구성하고, `<TestWrapper [theme={mockTheme}]><TestRenderer messages={messages} /></TestWrapper>`로 렌더링한 후 DOM 쿼리로 클래스 존재 여부를 검사한다.

| 테스트 케이스명 | 검증 동작 | 픽스처/모킹 |
|---|---|---|
| `should apply litTheme classes by default` | `theme` prop 없이 렌더링 시 `Button` 컴포넌트가 `litTheme.components.Button`의 키 중 하나 이상을 포함하고, `'test-button'` 클래스는 없음 | `Text(id='text-1', usageHint='body')` + `Button(id='btn-1', child='text-1', action.name='test')`, `createBeginRendering('btn-1')`. DOM 쿼리: `.a2ui-button button` |
| `should apply custom theme classes when theme prop is provided` | `theme={mockTheme}` 전달 시 `Button`에 `'test-button'` 클래스 존재, `litTheme.components.Button` 클래스는 없음 | 위와 동일 컴포넌트, `createMockTheme()`. DOM 쿼리: `.a2ui-button button` |
| `should apply custom theme to Card component` | `Card`의 내부 `section`에 `'test-card'` 클래스 존재 | `Text(id='text-1')` + `Card(id='card-1', child='text-1')`, `createBeginRendering('card-1')`, `createMockTheme()`. DOM 쿼리: `.a2ui-card > section` |
| `should apply custom theme to Text component` | `Text`의 내부 `section`에 `'test-text-all'` 클래스 존재 | `Text(id='text-1', usageHint='body')`, `createBeginRendering('text-1')`, `createMockTheme()`. DOM 쿼리: `.a2ui-text > section` |
| `should apply custom theme to Column component` | `Column`의 내부 `section`에 `'test-column'` 클래스 존재 | `Text(id='text-1')` + `Column(id='col-1', children.explicitList=['text-1'])`, `createBeginRendering('col-1')`, `createMockTheme()`. DOM 쿼리: `.a2ui-column > section` |
| `should apply custom theme to Row component` | `Row`의 내부 `section`에 `'test-row'` 클래스 존재 | `Text(id='text-1')` + `Row(id='row-1', children.explicitList=['text-1'])`, `createBeginRendering('row-1')`, `createMockTheme()`. DOM 쿼리: `.a2ui-row > section` |
| `should apply custom theme to TextField component` | `TextField`의 내부 `section`에 `'test-textfield-container'` 클래스 존재 | `TextField(id='tf-1', value.path='test.value', label.literalString='Name')`, `createBeginRendering('tf-1')`, `createMockTheme()`. DOM 쿼리: `.a2ui-textfield > section` |
| `should apply custom theme to CheckBox component` | `CheckBox`의 내부 `section`에 `'test-checkbox-container'` 클래스 존재 | `CheckBox(id='cb-1', value.path='test.checked', label.literalString='Accept')`, `createBeginRendering('cb-1')`, `createMockTheme()`. DOM 쿼리: `.a2ui-checkbox > section` |
| `should apply custom theme to Tabs component` | `Tabs`의 내부 `section`에 `'test-tabs-container'` 클래스 존재 | `Text(id='tab-content-1', text.literalString='Tab 1 Content')` + `Text(id='tab-content-2', text.literalString='Tab 2 Content')` + `Tabs(id='tabs-1', tabItems=[{title.literalString:'Tab 1', child:'tab-content-1'}, {title.literalString:'Tab 2', child:'tab-content-2'}])`, `createBeginRendering('tabs-1')`, `createMockTheme()`. DOM 쿼리: `.a2ui-tabs > section` |
| `should apply custom theme to Divider component` | `Divider`의 내부 `hr`에 `'test-divider'` 클래스 존재 | `Divider(id='div-1')`, `createBeginRendering('div-1')`, `createMockTheme()`. DOM 쿼리: `.a2ui-divider hr` |
| `should apply custom theme to Icon component` | `Icon`의 내부 `section`에 `'test-icon'` 클래스 존재 | `Icon(id='icon-1', name.literalString='home')`, `createBeginRendering('icon-1')`, `createMockTheme()`. DOM 쿼리: `.a2ui-icon > section` |
| `should apply custom theme to Slider component` | `Slider`의 내부 `section`에 `'test-slider-container'` 클래스 존재 | `Slider(id='slider-1', value.path='test.slider', min.literalNumber=0, max.literalNumber=100)`, `createBeginRendering('slider-1')`, `createMockTheme()`. DOM 쿼리: `.a2ui-slider > section` |
| `should apply custom theme to MultipleChoice component` | `MultipleChoice`의 내부 `section`에 `'test-multiplechoice-container'` 클래스 존재 | `MultipleChoice(id='mc-1', selections.literalArray=[], options=[{label.literalString:'Option A', value:'a'}, {label.literalString:'Option B', value:'b'}])`, `createBeginRendering('mc-1')`, `createMockTheme()`. DOM 쿼리: `.a2ui-multiplechoice > section` |

---

### `describe('ThemeContext') > describe('Theme Isolation')`

| 테스트 케이스명 | 검증 동작 | 픽스처/모킹 |
|---|---|---|
| `should not leak theme classes between different TestWrapper instances` | 목 테마로 렌더링 후 `unmount`하고, 기본 테마로 다시 렌더링했을 때 `'test-button'` 클래스가 없고 `litTheme` 클래스가 존재함 — 테마 상태가 인스턴스 간에 누출되지 않음을 검증 | 동일한 `messages`(Text + Button)를 두 번 렌더링. 첫 번째: `TestWrapper theme={mockTheme}` → `mockButton.classList.contains('test-button')` 참 확인 → `unmountMock()`. 두 번째: `TestWrapper` (theme prop 없음) → `defaultButton.classList.contains('test-button')` 가 `false`이고 `litTheme.components.Button` 키 중 하나를 포함하는지 확인 |

## 동작 흐름

각 테스트는 서버 메시지 배열을 구성 → `TestWrapper`로 `A2UIProvider` 컨텍스트 제공 → `TestRenderer`가 `useEffect`에서 메시지 처리 → `A2UIRenderer`가 루트 컴포넌트 렌더링 → DOM에서 CSS 클래스 존재 여부로 테마 적용 결과를 단언한다. `Theme Isolation` 테스트는 두 번의 독립적인 `render` 호출을 통해 컨텍스트 격리 여부를 확인한다.
