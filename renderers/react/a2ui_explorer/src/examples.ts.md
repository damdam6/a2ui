# renderers/react/a2ui_explorer/src/examples.ts

## 개요

생성된 예제 모듈 레지스트리를 소비하여 `DemoItem` 배열로 변환하는 유틸리티 모듈이다. JSON 파일명을 사람이 읽기 좋은 제목으로 변환하고, 예제 데이터에서 메시지 배열과 설명을 추출하는 책임을 담당한다.

## 의존성

### 외부 패키지
- `@a2ui/web_core/v0_9` — `A2uiMessage` 타입

### 저장소 내부 모듈
- [`./generated/examples-list`](./generated/examples-list.ts.md) — `exampleModules` 상수, `ExampleModule` 타입, `ExampleData` 타입

## Exports

- `DemoItem` (interface) — 예제 데이터 항목 타입
- `processExampleModules` (함수) — 모듈 레지스트리를 `DemoItem[]`으로 변환 (테스트 가시성 목적으로 export)
- `getDemoItems` (함수) — 전체 데모 항목 목록 반환

## 상세 명세

### `DemoItem` (interface)

| 필드 | 타입 | 설명 |
|---|---|---|
| `id` | `string` | 고유 식별자 (파일명에서 `.json` 제거) |
| `title` | `string` | 파일명 기반 사람이 읽기 좋은 제목 |
| `filename` | `string` | 원본 JSON 파일명 |
| `description` | `string` | 예제 설명 또는 폴백 소스 문자열 |
| `messages` | `A2uiMessage[]` | 이 데모에서 처리할 A2UI 메시지 배열 |

### `processExampleModules(modules: Record<string, ExampleModule>): DemoItem[]`

**@internal @visibleForTesting**

모듈 레지스트리 맵을 받아 `DemoItem[]`을 반환한다.

1. `Object.entries(modules)`를 파일명 기준 `localeCompare` 정렬로 처리한다.
2. 각 `[filename, data]` 쌍에 대해:
   - `data.default`에서 `extractMessagesAndDescription`을 호출하여 메시지 배열과 설명을 추출한다.
   - `id`는 `filename.replace('.json', '')`으로 생성한다.
   - `title`은 `filenameToTitle(filename)`으로 생성한다.
   - 에러 발생 시 `console.error`로 기록하고 해당 항목을 건너뛴다.
3. 수집된 `DemoItem[]`을 반환한다.

### `extractMessagesAndDescription(jsonData: ExampleData | A2uiMessage[], filename: string): [A2uiMessage[], string]` (비공개)

예제 JSON 데이터에서 메시지 배열과 설명을 추출한다.

- `jsonData`가 배열이면 그것을 `messages`로 사용하고 설명은 `Source: ${filename}`으로 폴백한다.
- `jsonData`가 객체이면 `jsonData.messages || []`를 메시지로, `jsonData.description || 폴백`을 설명으로 사용한다.
- 메시지 배열이 비어 있으면 `console.warn`으로 경고를 출력한다.
- `[messages, description]` 튜플을 반환한다.

### `filenameToTitle(filename: string): string` (비공개)

파일명을 사람이 읽기 좋은 제목으로 변환한다.

변환 단계:
1. `.json` 제거
2. 선두 숫자 접두사 제거: `/^[0-9]+_/` 패턴 제거
3. `-` 와 `_`를 공백으로 교체
4. 단어 첫 글자를 대문자로 변환 (`/\b\w/g` 패턴)

예: `02_email-compose.json` → `Email Compose`

### `getDemoItems(): DemoItem[]`

`processExampleModules(exampleModules)`를 호출하여 전체 데모 목록을 반환한다. 인수 없음.

## 동작 흐름

`getDemoItems` → `processExampleModules(exampleModules)` → 각 파일에 대해 `extractMessagesAndDescription` + `filenameToTitle` 호출 → `DemoItem[]` 반환. 최종적으로 `App.tsx`가 이 배열을 모듈 레벨에서 한 번 호출하여 `demoItems` 상수로 캐시한다.
