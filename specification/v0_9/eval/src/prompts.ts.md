# specification/v0_9/eval/src/prompts.ts

## 개요

평가 파이프라인이 각 LLM 모델에 전송할 테스트 프롬프트 목록을 정의하는 파일이다. `TestPrompt` 인터페이스로 프롬프트의 구조를 명시하고, `prompts` 배열에 A2UI 스펙의 다양한 UI 패턴을 검증하는 43개의 구체적인 프롬프트 항목을 담는다. 각 프롬프트는 고유한 `name`(식별자), `description`(짧은 설명), `promptText`(LLM에 전달하는 실제 지시문)로 구성된다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈
없음

## Exports

| 이름 | 종류 | 설명 |
|------|------|------|
| `TestPrompt` | 인터페이스 | 단일 테스트 프롬프트의 형태 |
| `prompts` | 상수 배열 (`TestPrompt[]`) | 평가에 사용할 모든 테스트 프롬프트 |

## 상세 명세

### `TestPrompt` 인터페이스 (export)

| 필드 | 타입 | 설명 |
|------|------|------|
| `name` | `string` | 프롬프트 고유 식별자 (파일명 등에 사용) |
| `description` | `string` | 프롬프트가 테스트하는 내용의 짧은 설명 |
| `promptText` | `string` | LLM에 전달하는 실제 지시 텍스트 |

### `prompts: TestPrompt[]` (export)

43개의 `TestPrompt` 항목을 담은 배열. 각 항목의 `name`과 테스트 목적은 다음과 같다.

| `name` | 테스트 목적 |
|--------|-------------|
| `deleteSurface` | `DeleteSurface` 메시지 생성 |
| `dogBreedGenerator` | `createSurface` + `updateComponents` 조합, 카드·리스트·폼·`ChoicePicker` 복합 레이아웃 |
| `loginForm` | 로그인 폼, 데이터 바인딩, 버튼 액션 `event` 구조 |
| `productGallery` | 리스트 템플릿, 데이터 바인딩, 버튼 `context` |
| `productGalleryData` | `updateDataModel` 메시지, `/products` 경로 중첩 맵 구조 |
| `settingsPage` | `Tabs`, `Modal`, 트리거·콘텐츠 구조 |
| `updateDataModel` | `updateDataModel`로 `/user/name`, `/user/email` 설정 |
| `animalKingdomExplorer` | 3탭 + 중첩 `Card` 계층 구조 (클래스→목→종) |
| `recipeCard` | `Row`·`Column` 분할, 정적 `List` |
| `musicPlayer` | `Slider`, 자식 `Text`를 가진 `Button` 패턴 |
| `weatherForecast` | `Divider`, 5일 예보 `List` |
| `surveyForm` | `ChoicePicker` (mutuallyExclusive, multipleSelection) |
| `flightBooker` | `DateTimeInput`, `Slider` |
| `dashboard` | 통계 카드 `Row` |
| `contactCard` | 루트가 `Card`인 구조 |
| `calendarEventCreator` | `DateTimeInput`에 리터럴 빈 문자열 초기화 |
| `checkoutPage` | 다중 `TextField` 폼 |
| `socialMediaPost` | 소셜 포스트 카드 레이아웃 |
| `eCommerceProductPage` | `Row` 2열 레이아웃, 다중 `ChoicePicker` |
| `interactiveDashboard` | 필터 카드 + 메트릭 카드 + 플레이스홀더 이미지 |
| `travelItinerary` | 중첩 `List` (외부 카드 리스트 → 내부 활동 리스트), `CheckBox` |
| `kanbanBoard` | 3열 칸반, `Column`·`Card` 구조 |
| `videoCallInterface` | `Video` 컴포넌트 |
| `fileBrowser` | `Icon` 컴포넌트 (folder, attachFile) |
| `chatRoom` | 채팅 UI, `TextField` + Send 버튼 |
| `fitnessTracker` | 카드 행, `Slider`, 정적 `List` |
| `smartHome` | `Grid`를 `Column`+`Row` 조합으로 표현, `CheckBox`·`Slider` |
| `restaurantMenu` | `Tabs` + 분리된 `Row` 컴포넌트 ID 참조 패턴 |
| `newsAggregator` | 루트 `Column`, `Text`와 `List`가 형제 관계 |
| `photoEditor` | 이미지 + 버튼 행 + `Slider` |
| `triviaQuiz` | `ChoicePicker` (mutuallyExclusive) 답변 선택 |
| `simpleCalculator` | `Card`→`Column`→`Row` 버튼 그리드 |
| `jobApplication` | `ChoicePicker` (mutuallyExclusive) + `TextField` 폼 |
| `courseSyllabus` | 중첩 `Card`+`List` 모듈 구조 |
| `stockWatchlist` | 정적 `Row` 항목 3개 |
| `podcastEpisode` | `Slider` value 0 초기화, 정적 컴포넌트 |
| `hotelSearchResults` | `Row`·`Image`·`Column`·`Button` 카드 |
| `notificationCenter` | `Dismiss` 버튼이 있는 알림 카드 리스트 |
| `nestedDataBinding` | 3개 메시지 스트림, 깊이 3단계 데이터 바인딩(`projects`→`tasks`→`subtasks`), `updateDataModel` 포함 |
| `profileEditor` | multiline `TextField` |
| `cinemaSeatSelection` | `CheckBox` 그리드 (3행 × 4열) |
| `flashcardApp` | `Card`+`Divider`+버튼 행 패턴 |
| `clientSideValidation` | `TextField` regex 유효성 검사 (`^[a-zA-Z0-9]{3,}$`) |
| `standardFunctions` | `pluralize` 함수 호출 (returnType `string`, zero/one/other 옵션) |
| `openUrlAction` | 버튼 액션에 `openUrl` 클라이언트 함수 호출 |
| `nestedLayoutRecursive` | 5단계 깊이 중첩 레이아웃 (Card→Column→Row→List→Text) |

각 `promptText`는 LLM이 A2UI v0.9 스펙에 맞는 JSON 메시지 스트림을 생성하도록 지시하며, `surfaceId`, 컴포넌트 타입, 데이터 경로, 액션 구조 등을 구체적으로 명시한다.

## 동작 흐름

모듈 로드 시 `prompts` 배열이 즉시 생성된다. 평가 파이프라인의 메인 루프에서 `prompts`를 import하여 각 항목을 모든 모델과 교차하며 실행한다. `TestPrompt`의 `name` 필드는 결과 파일 이름 등 식별자로 활용된다.
