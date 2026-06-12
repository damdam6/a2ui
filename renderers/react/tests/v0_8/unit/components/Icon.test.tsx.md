# renderers/react/tests/v0_8/unit/components/Icon.test.tsx

## 개요

A2UI 명세를 따르는 `Icon` 컴포넌트의 단위 테스트 모음이다. A2UI 아이콘 이름이 Material Symbols 폰트용 snake_case로 변환되는 매핑 규칙, `.g-icon` 클래스를 가진 `span` 요소 렌더링, 빈 이름·미지정 이름 처리, `defaultTheme`·`litTheme` 테마별 동작을 검증한다.

## 의존성

### 외부 패키지
- `vitest` — `describe`, `it`, `expect`, `beforeEach`
- `@testing-library/react` — `render`, `screen`
- `react` — `React`

### 저장소 내부 모듈
- [`../../utils`](../../utils/index.ts.md) — `TestWrapper`, `TestRenderer`, `createSimpleMessages`
- [`../../../../src/v0_8`](../../../../src/v0_8/index.ts.md) — `litTheme`, `defaultTheme`

## Exports

이 파일은 아무것도 export하지 않는다.

## 상세 명세 (테스트 케이스)

### describe: `Icon Component`

#### describe: `Basic Rendering`

- **`should render an icon with literal name`**
  - 검증: `name: {literalString: 'home'}`으로 렌더링하면 `.a2ui-surface` 요소가 DOM에 존재하고 `innerHTML`이 비어 있지 않다.

- **`should render icon with empty string name gracefully`**
  - 검증: `name: {literalString: ''}`으로 렌더링하면 에러 없이 `.a2ui-surface`가 존재한다. 주석에 따르면 빈 이름일 때 아이콘은 `null`을 반환(렌더링 안 됨).

- **`should render with search icon`**
  - 검증: `name: {literalString: 'search'}` → `.a2ui-surface` 존재.

- **`should render with settings icon`**
  - 검증: `name: {literalString: 'settings'}` → `.a2ui-surface` 존재.

#### describe: `Icon Name Mapping`

camelCase A2UI 이름이 Material Symbols용 snake_case로 변환되는 규칙을 파라미터화 테스트로 검증한다.

테스트 데이터 배열 `iconMappings` (10개 항목):

| a2ui (입력) | expected (변환 후) |
|---|---|
| `'home'` | `'home'` |
| `'search'` | `'search'` |
| `'settings'` | `'settings'` |
| `'favorite'` | `'favorite'` |
| `'delete'` | `'delete'` |
| `'shoppingCart'` | `'shopping_cart'` |
| `'accountCircle'` | `'account_circle'` |
| `'notifications'` | `'notifications'` |
| `'mail'` | `'mail'` |
| `'lock'` | `'lock'` |

- **`should render "${a2ui}" icon as "${expected}"` (파라미터화)**
  - 검증: A2UI 이름으로 Icon을 렌더링하면 `.g-icon` 요소가 DOM에 존재하고, `textContent`가 snake_case 변환된 `expected` 문자열과 일치한다.
  - `iconMappings.forEach`로 10개의 개별 테스트를 동적 생성한다.

#### describe: `Theme Support`

- **`should apply default theme classes`**
  - 픽스처: `TestWrapper`에 `theme={defaultTheme}` 전달, `name: 'home'`.
  - 검증: `section` 요소와 `.g-icon` 요소가 모두 DOM에 존재한다. 주석에 따르면 `defaultTheme`(litTheme)의 Icon 클래스는 비어 있을 수 있다.

- **`should apply lit theme classes with container/element structure`**
  - 픽스처: `TestWrapper`에 `theme={litTheme}` 전달, `name: 'home'`.
  - 검증: `.g-icon` 요소가 DOM에 존재한다.

#### describe: `Material Symbols Integration`

- **`should render icon using Material Symbols font`**
  - 검증: `name: 'home'` → `.g-icon` 존재, `textContent === 'home'`. Material Symbols 폰트는 span 텍스트 내용을 아이콘 이미지로 표시한다.

- **`should convert camelCase icon names to snake_case`**
  - 검증: `name: 'shoppingCart'` → `.g-icon`의 `textContent === 'shopping_cart'`.

#### describe: `Unknown Icons`

- **`should render unknown icon names as-is for Material Symbols`**
  - 검증: `name: 'unknownIconName12345'`처럼 알 수 없는 camelCase 이름을 전달하면 `.g-icon`의 `textContent`가 `'unknown_icon_name12345'`(snake_case 변환 결과)이다. 폰트가 알 수 없는 이름을 그대로 표시하는 방식으로 폴백된다.

## 동작 흐름

각 테스트는 `createSimpleMessages`로 `Icon` 컴포넌트 메시지를 생성하고 `TestRenderer`로 렌더링한다. 아이콘 이름은 렌더러 내부에서 camelCase → snake_case 변환을 거친 뒤 `<span class="g-icon">` 요소의 텍스트 내용으로 출력된다. 테마 테스트는 `TestWrapper`의 `theme` prop으로 `litTheme`/`defaultTheme`를 주입하여 테마별 클래스 적용 여부를 확인한다. `beforeEach`는 import되었으나 현재 사용되지 않는다.
