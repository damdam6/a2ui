# renderers/react/a2ui_explorer/scripts/generate-examples.mjs

## 개요

빌드 전처리 스크립트로, 사양(specification) 디렉터리에 있는 v0_9 basic catalog 예제 JSON 파일들을 스캔하여 정적 TypeScript 모듈(`src/generated/examples-list.ts`)을 자동 생성한다. Vite의 `import.meta.glob` API에 의존하지 않고도 React 탐색기와 통합 테스트 모두에서 예제 파일을 사용할 수 있도록 표준 ES import 구문 기반의 번들을 만든다.

## 의존성

### 외부 패키지
- `fs` (Node.js 내장) — 파일 시스템 읽기/쓰기
- `path` (Node.js 내장) — 경로 조작

### 저장소 내부 모듈
없음 (스크립트는 파일 시스템만 조작함)

## Exports

기본 export 없음. 스크립트가 직접 실행(`generateExamplesBundle()` 호출)된다.

## 상세 명세

### 상수

- `SPEC_EXAMPLES_DIR`: `import.meta.dirname`을 기준으로 `../../../../specification/v0_9/catalogs/basic/examples`를 절대 경로로 resolve한 값. 예제 JSON 파일들이 위치하는 입력 디렉터리이다.
- `OUT_FILE`: `import.meta.dirname`을 기준으로 `../src/generated/examples-list.ts`를 절대 경로로 resolve한 값. 생성될 TypeScript 모듈의 출력 경로이다.

### `generateExamplesBundle()` — `void`

매개변수 없음. 스크립트 최하단에서 즉시 호출된다.

**동작 단계:**

1. `SPEC_EXAMPLES_DIR`이 존재하지 않으면 오류 메시지를 출력하고 `process.exit(1)`로 종료한다.
2. `fs.readdirSync`로 디렉터리를 읽고 `.json`으로 끝나는 파일만 필터링한 뒤 알파벳 순으로 정렬한다.
3. 각 파일에 대해 다음을 수행한다:
   - `src/generated/examples-list.ts` 위치에서 사양 디렉터리까지의 상대 경로(`../../../../../specification/v0_9/catalogs/basic/examples/<file>`)를 계산한다.
   - 변수명을 `example_0`, `example_1`, … 형태로 생성한다.
   - `imports` 배열에 `import example_N from '<relativePath>';` 형태의 import 문을 추가한다.
   - `entries` 배열에 `'<filename>': { default: example_N }` 형태의 객체 항목을 추가한다.
4. 생성할 파일의 내용을 템플릿 문자열로 조립한다. 파일에는 다음이 포함된다:
   - 자동 생성 주석 헤더 (재생성 방법 안내 포함)
   - `@a2ui/web_core/v0_9`에서 `A2uiMessage` 타입 import
   - 수집된 import 문들
   - `ExampleData` 인터페이스: `messages?: A2uiMessage[]`, `description?: string`, `name?: string` 필드
   - `ExampleModule` 인터페이스: `default: ExampleData | A2uiMessage[]` 필드
   - `exampleModules: Record<string, ExampleModule>` 상수 (수집된 entries로 구성)
5. 출력 디렉터리가 없으면 `fs.mkdirSync(outDir, {recursive: true})`로 생성한다.
6. `fs.writeFileSync`로 파일을 UTF-8 인코딩으로 기록하고 성공 메시지를 콘솔에 출력한다.

## 동작 흐름

스크립트는 `node generate-examples.mjs`(또는 `yarn generate-examples`)로 실행된다. 입력 디렉터리 존재 여부 검사 → JSON 파일 목록 수집 → import 문과 레코드 항목 생성 → 출력 디렉터리 보장 → TypeScript 모듈 파일 기록 순서로 진행된다. 생성된 파일은 커밋에 포함되며, 예제 JSON 추가/삭제/이름 변경 시 반드시 재실행해야 한다.
