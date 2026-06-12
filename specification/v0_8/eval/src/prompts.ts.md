# specification/v0_8/eval/src/prompts.ts

## 개요

이 파일은 평가 시스템에서 사용할 테스트 프롬프트 목록과 각 프롬프트에 대한 기대 결과 검증기(matcher)를 정의한다. `TestPrompt` 인터페이스와 `prompts` 배열을 export하여 `index.ts`가 각 모델에 대해 무엇을 생성하고 어떻게 검증할지 결정하는 데 사용한다. 모든 프롬프트는 동일한 JSON 스키마 파일(`server_to_client_with_standard_catalog.json`)을 사용한다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈
- [`./basic_schema_matcher`](./basic_schema_matcher.ts.md) — `BasicSchemaMatcher`
- [`./message_type_matcher`](./message_type_matcher.ts.md) — `MessageTypeMatcher`
- [`./schema_matcher`](./schema_matcher.ts.md) — `SchemaMatcher`
- [`./surface_update_schema_matcher`](./surface_update_schema_matcher.ts.md) — `SurfaceUpdateSchemaMatcher`

## Exports

| 이름 | 종류 | 설명 |
|------|------|------|
| `TestPrompt` | interface | 단일 테스트 프롬프트의 구조 타입 |
| `prompts` | `TestPrompt[]` 상수 | 전체 평가 테스트 프롬프트 목록 |

## 상세 명세

### `TestPrompt` (interface)

다섯 필드를 가진다.

- `name: string` — 프롬프트 고유 식별자. CLI `--prompt` 필터와 출력 파일명에 사용된다.
- `description: string` — 프롬프트가 테스트하는 시나리오의 간단한 설명.
- `schemaPath: string` — `__dirname` 기준 상대 경로로, 생성에 사용할 JSON Schema 파일 경로.
- `promptText: string` — LLM에 전달할 실제 텍스트 프롬프트.
- `matchers: SchemaMatcher[]` — 생성 결과를 검증할 매처 인스턴스 배열.

### `schemaPath` (내부 상수)

`'../../json/server_to_client_with_standard_catalog.json'` — 모든 프롬프트가 공유하는 스키마 파일 경로.

### `prompts` (상수, `TestPrompt[]`)

총 20개의 테스트 프롬프트를 포함한다. 각 항목 요약:

| `name` | 메시지 타입 | 핵심 검증 항목 |
|--------|------------|---------------|
| `deleteSurface` | `deleteSurface` | `surfaceId`가 `'dashboard-surface-1'`인지 |
| `dogBreedGenerator` | `surfaceUpdate` | `Column`, `Image`, `TextField`(dog breed name, number of legs), `Button`(Generate) |
| `loginForm` | `surfaceUpdate` | `Heading`(Login), `TextField`(username, password), `CheckBox`(Remember Me), `Button`(Sign In) |
| `productGallery` | `surfaceUpdate` | `Column`, `Card`, `Image`, `Text`, `Button`(Add to Cart) |
| `productGalleryData` | `dataModelUpdate` | `path`가 `'/products'`, 첫 번째 항목 `key`와 `valueMap` 존재 |
| `settingsPage` | `surfaceUpdate` | `TextField`(name), `CheckBox`(Enable email notifications), 다수의 `Button` |
| `dataModelUpdate` | `dataModelUpdate` | 최상위 타입만 검사 |
| `uiRoot` | `beginRendering` | 최상위 타입만 검사 |
| `animalKingdomExplorer` | `surfaceUpdate` | `Heading`, `TextField`(Search...), 동물계 계층의 22개 `Text` 컴포넌트 |
| `recipeCard` | `surfaceUpdate` | `Heading`(Classic Lasagna, Ingredients, Instructions), `Image`, `Column`, `Button`(Watch Video Tutorial) |
| `musicPlayer` | `surfaceUpdate` | `Column`, `Image`, `Text`(Bohemian Rhapsody, Queen), `Slider`, 3개 `Button` |
| `weatherForecast` | `surfaceUpdate` | `Heading`(New York), `Text`(68°F), `Image`, `List` |
| `surveyForm` | `surfaceUpdate` | `Heading`, `MultipleChoice`(Excellent), `CheckBox`(Product Quality), `TextField`, `Button` |
| `flightBooker` | `surfaceUpdate` | `Heading`(Book a Flight), 2개 `TextField`, `DateTimeInput`, `CheckBox`, `Button` |
| `dashboard` | `surfaceUpdate` | `Heading`(Sales Dashboard), `Column`, 6개 `Text` 컴포넌트 |
| `contactCard` | `surfaceUpdate` | `Column`, `Image`, 3개 `Text`, `Button`(View on Map) |
| `calendarEventCreator` | `surfaceUpdate` | `Heading`(New Event), `TextField`, `DateTimeInput`, `CheckBox`, 2개 `Button` |
| `checkoutPage` | `surfaceUpdate` | `Heading`(Checkout), 4개 `TextField`, `Text`(Total: $99.99), `Button`(Place Order) |
| `socialMediaPost` | `surfaceUpdate` | `Column`, 2개 `Text`, `Image`, 3개 `Button` |
| `eCommerceProductPage` | `surfaceUpdate` | `Heading`, `Text`($299.99), `Image`, 2개 `MultipleChoice`, `Button`(Add to Cart) |
| `interactiveDashboard` | `surfaceUpdate` | `Heading`(Company Dashboard), `DateTimeInput`, `Button`(Apply Filters), 다수 `Heading`/`Text` |
| `travelItinerary` | `surfaceUpdate` | `Heading`(Paris Adventure, 3개 Day heading), `Column`, `CheckBox`, 3개 활동 `Text` |

`dogBreedGenerator`, `loginForm`, `settingsPage`, `animalKingdomExplorer` 등의 `TextField` 매처는 `caseInsensitive: true`로 생성된다(생성자 4번째 인자 `true`).

## 동작 흐름

모듈 로드 시 `prompts` 배열이 즉시 초기화된다. `index.ts`에서 이 배열을 참조하여 CLI `--prompt` 옵션으로 이름 접두사 필터링한 뒤, 각 프롬프트의 `schemaPath`로 JSON 스키마를 읽고 `promptText`를 LLM에 전달한다. 생성 결과는 `matchers` 배열의 각 매처를 순서대로 실행하여 검증된다.
