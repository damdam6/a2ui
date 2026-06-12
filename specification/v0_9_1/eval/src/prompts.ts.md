# specification/v0_9_1/eval/src/prompts.ts

## 개요

이 파일은 평가에 사용할 테스트 프롬프트의 목록을 정의한다. `TestPrompt` 인터페이스와 `prompts` 배열을 내보내며, 각 항목은 고유 식별자(`name`), 설명(`description`), 모델에게 전달할 실제 지시문(`promptText`)으로 구성된다. 평가 파이프라인의 생성 단계에서 이 배열을 순회하여 각 모델에 프롬프트를 전달하고 응답을 수집한다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈
없음

## Exports

| 이름 | 종류 |
|---|---|
| `TestPrompt` | 인터페이스 |
| `prompts` | `TestPrompt[]` 상수 |

## 상세 명세

### `TestPrompt` 인터페이스 (export)

| 필드 | 타입 | 설명 |
|---|---|---|
| `name` | `string` | 프롬프트의 고유 식별자. 결과 파일명 등에 사용됨 |
| `description` | `string` | 프롬프트의 목적에 대한 짧은 설명 |
| `promptText` | `string` | 모델에게 전달할 실제 지시문 전체 |

### `prompts: TestPrompt[]` (export)

총 42개의 테스트 프롬프트가 등록된 배열. 각 프롬프트의 `name`과 기능 분류는 다음과 같다.

**기본 메시지 타입 테스트**
- `'deleteSurface'` — `DeleteSurface` 메시지 생성 (surface `'dashboard-surface-1'`)
- `'updateDataModel'` — `updateDataModel` 메시지 생성 (사용자 로그인 후 프로필 데이터 갱신)
- `'productGalleryData'` — `createSurface` + `updateDataModel` 메시지 조합

**단순 레이아웃**
- `'loginForm'` — 로그인 폼 (username, password, 체크박스, 버튼)
- `'contactCard'` — 연락처 카드 (Card > Row > Image + Column 구조)
- `'photoEditor'` — 사진 편집 UI (Image, 버튼 Row, Slider)
- `'triviaQuiz'` — 퀴즈 카드 (Text, ChoicePicker, Button)
- `'jobApplication'` — 취업 지원 폼
- `'openUrlAction'` — `openUrl` 클라이언트 함수 액션을 가진 버튼

**리스트 / 갤러리**
- `'productGallery'` — 상품 갤러리 (List, template, Card, Image, Button)
- `'hotelSearchResults'` — 호텔 검색 결과 (List of Card)
- `'newsAggregator'` — 뉴스 피드 (Column > Text + List of Card)
- `'stockWatchlist'` — 주식 목록 (List of Row)
- `'notificationCenter'` — 알림 목록 (List of Card with Dismiss 버튼)
- `'restaurantMenu'` — 레스토랑 메뉴 (Tabs + List of Row)
- `'fileBrowser'` — 파일 탐색기 (List of Row with Icon + Text)

**폼 및 입력**
- `'surveyForm'` — 고객 피드백 설문 (ChoicePicker × 2, TextField, Button)
- `'flightBooker'` — 항공편 검색 폼 (DateTimeInput, Slider)
- `'calendarEventCreator'` — 캘린더 이벤트 생성 폼 (DateTimeInput 리터럴 빈 문자열)
- `'checkoutPage'` — 간소화된 e-commerce 결제 페이지
- `'clientSideValidation'` — 정규식 유효성 검사가 있는 등록 폼 (`^[a-zA-Z0-9]{3,}$`)
- `'cinemaSeatSelection'` — 좌석 선택 그리드 (Column of Row of CheckBox)
- `'profileEditor'` — 프로필 편집 폼

**대시보드 / 복합 UI**
- `'dashboard'` — 단순 대시보드 (Row of Card × 3)
- `'settingsPage'` — 설정 페이지 (Tabs + Modal)
- `'interactiveDashboard'` — 필터 카드 + 메트릭 카드 + 차트 카드
- `'kanbanBoard'` — 칸반 보드 (Row of Column of Card)
- `'smartHome'` — 스마트 홈 제어판 (Column of Row of Card, CheckBox, Slider)
- `'fitnessTracker'` — 피트니스 트래커 대시보드 (Card × 3, Slider, List)

**복잡한 구조 / 중첩 테스트**
- `'dogBreedGenerator'` — 세로 목록 + 카드 + Form (List, ChoicePicker multipleSelection)
- `'animalKingdomExplorer'` — 동물 계층 탐색기 (Tabs × 3, 중첩 Card)
- `'recipeCard'` — 레시피 카드 (Row of Column, List)
- `'courseSyllabus'` — 강의 계획서 (List of Card with nested List)
- `'travelItinerary'` — 여행 일정 (List of Card with nested List of Row with CheckBox)
- `'nestedDataBinding'` — 깊은 데이터 바인딩 (List > template > List > template > List)
- `'nestedLayoutRecursive'` — 5단계 중첩 레이아웃 (Card > Column > Row > List > Text)

**특수 컴포넌트**
- `'musicPlayer'` — 음악 플레이어 (Card, Slider, Row of Button)
- `'weatherForecast'` — 날씨 예보 (Text, Row, Divider, List)
- `'podcastEpisode'` — 팟캐스트 플레이어 (Card, Slider, Row of Button)
- `'videoCallInterface'` — 화상 통화 UI (Video 컴포넌트 포함)
- `'chatRoom'` — 채팅방 UI (Column of Card, Row with TextField + Button)
- `'eCommerceProductPage'` — 상품 상세 페이지 (Row, ChoicePicker mutuallyExclusive × 2)

**함수 / 액션 테스트**
- `'standardFunctions'` — `pluralize` 함수 호출 (`returnType: 'string'`, `/cart/count` 바인딩)
- `'simpleCalculator'` — 계산기 (Card > Column > Text + Column of Row of Button)
- `'socialMediaPost'` — 소셜 미디어 게시물 (Card, Row of Button × 3)
- `'flashcardApp'` — 단어장 앱 (Card > Column with Divider)

모든 `promptText`는 a2ui 프로토콜의 `createSurface`, `updateComponents`, `updateDataModel`, `deleteSurface` 메시지 형식으로 JSON을 생성하도록 모델에게 요청한다.

## 동작 흐름

이 파일은 순수하게 선언적이다. 모듈 로드 시 `prompts` 배열이 메모리에 올라가며, 평가 진입점이 이 배열을 가져와 `modelsToTest`와 교차 순회하여 각 (모델, 프롬프트) 쌍에 대해 생성 요청을 수행한다.
