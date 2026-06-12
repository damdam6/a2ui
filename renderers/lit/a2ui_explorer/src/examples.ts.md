# renderers/lit/a2ui_explorer/src/examples.ts

## 개요

specification 디렉토리의 예시 JSON 파일들을 로드하고 파싱하여 갤러리가 소비할 수 있는 `DemoItem` 배열로 변환하는 모듈이다. JSON이 직접 메시지 배열인 경우와 메타데이터를 포함하는 객체인 경우 모두 처리하며, `createSurface` 메시지가 없는 예시에는 합성된 메시지를 자동으로 주입한다. 공개 API는 `getDemoItems()` 함수와 `DemoItem` 인터페이스다.

## 의존성

### 외부 패키지
- `@a2ui/lit/v0_9` — `basicCatalog` (catalog ID를 합성 `createSurface` 메시지에 사용)
- `@a2ui/web_core/v0_9` — `A2uiMessage`, `CreateSurfaceMessage` 타입

### 저장소 내부 모듈
- [`./generated/examples-list`](./generated/examples-list.ts.md) — `ExampleData`, `ExampleModule` 인터페이스, `exampleModules` 상수 (빌드 시 자동 생성된 파일)

## Exports

- `DemoItem` (인터페이스) — 갤러리 항목의 구조를 정의
- `getDemoItems` (함수) — `DemoItem[]`을 반환하는 공개 함수

## 상세 명세

### `DemoItem` 인터페이스

갤러리의 단일 항목을 나타낸다.

| 필드 | 타입 | 설명 |
|------|------|------|
| `id` | `string` | surface의 고유 식별자 (보통 surfaceId) |
| `title` | `string` | 파일명에서 파생된 사람이 읽을 수 있는 제목 |
| `filename` | `string` | 원본 파일명 |
| `description` | `string` | 예시의 설명 또는 fallback 소스 문자열 |
| `messages` | `A2uiMessage[]` | 처리될 A2UI 메시지 배열 |

### `getDemoItems(): DemoItem[]`

매개변수 없음. `DemoItem[]`을 반환한다.

동작 단계:
1. `getSortedExampleEntries()`를 호출하여 정렬된 `[filename, ExampleModule]` 튜플 배열을 얻는다.
2. 각 항목에 대해 try-catch 블록 안에서:
   - `data.default`에서 `jsonData`를 추출한다.
   - `extractMessagesAndDescription(jsonData, filename)`으로 `messages`와 `description`을 분리한다.
   - `ensureCreateSurfaceMessage(filename, messages)`로 `surfaceId`를 확보하고 필요 시 메시지 배열을 수정한다.
   - `items` 배열에 `DemoItem` 객체를 push한다.
   - 오류 발생 시 `console.error`로 기록하고 해당 파일을 건너뛴다.
3. 파싱된 항목이 없으면 `console.warn`을 출력한다.
4. 완성된 `items` 배열을 반환한다.

### `ensureCreateSurfaceMessage(filename: string, messages: A2uiMessage[]): string` (비공개)

`createSurface` 메시지의 존재를 보장하고 `surfaceId`를 반환한다.

동작 단계:
1. `surfaceId`를 `filename.replace('.json', '')`로 초기화한다.
2. `messages` 배열에서 `'createSurface' in message` 조건을 만족하는 `CreateSurfaceMessage`를 찾는다.
3. 존재하면 해당 메시지의 `createSurface.surfaceId`를 `surfaceId`로 사용한다.
4. 존재하지 않으면 `version: 'v0.9'`, `createSurface: { surfaceId, catalogId: basicCatalog.id }` 구조의 메시지를 `messages.unshift`로 배열 앞에 삽입한다. **주의: 이 메서드는 `messages` 배열을 직접 변경(mutate)한다.**
5. `surfaceId`를 반환한다.

### `getSortedExampleEntries(): [string, ExampleModule][]` (비공개)

`exampleModules`의 엔트리를 파일 경로 기준으로 알파벳 오름차순 정렬하여 반환한다. `Object.entries(exampleModules).sort((a, b) => a[0].localeCompare(b[0]))` 로 구현된다.

### `extractMessagesAndDescription(jsonData: ExampleData | A2uiMessage[], filename: string): [A2uiMessage[], string]` (비공개)

JSON 데이터에서 메시지 배열과 설명 문자열을 추출한다.

동작 단계:
1. `messages`를 빈 배열로, `description`을 `` `Source: ${filename}` ``로 초기화한다.
2. `jsonData`가 배열이면 직접 `messages`로 사용한다.
3. 배열이 아니면(래핑된 `ExampleData` 객체) `jsonData.messages || []`를 `messages`로, `jsonData.description || description`을 `description`으로 사용한다.
4. `messages`가 비어 있으면 `console.warn`으로 경고한다.
5. `[messages, description]` 튜플을 반환한다.

### `filenameToTitle(filename: string): string` (비공개)

파일명을 사람이 읽을 수 있는 제목으로 변환한다.

변환 단계:
1. `.json` 확장자 제거
2. 선행 숫자 접두사 패턴(`/^[0-9]+_/`) 제거 — 예: `02_` 제거
3. 하이픈(`-`)과 언더스코어(`_`)를 공백으로 교체
4. 각 단어의 첫 글자를 대문자로 변환 (`/\b\w/g` 패턴 사용)

예시: `"02_email-compose.json"` → `"Email Compose"`

## 동작 흐름

모듈이 import되면 `exampleModules`(자동 생성된 정적 매핑)에서 엔트리를 읽는다. `getDemoItems()`가 호출되면 정렬 → 파싱 → 보정(createSurface 주입) → 변환 파이프라인을 거쳐 갤러리 컴포넌트가 소비할 수 있는 `DemoItem` 배열을 반환한다.
