# renderers/lit/a2ui_explorer/scripts/generate-examples.mjs

## 개요

이 Node.js 스크립트는 specification 디렉토리에 있는 v0.9 basic catalog 예시 JSON 파일들을 정적으로 번들링한 TypeScript 모듈(`src/generated/examples-list.ts`)을 생성한다. Vite 전용 API인 `import.meta.glob`를 사용하지 않고도 Karma 같은 표준 ES 컴파일러 환경에서도 예시 파일들을 런타임에 접근할 수 있도록 하기 위해 존재한다. `yarn generate-examples` 명령으로 실행하며, 예시 JSON 파일 추가·삭제·이름 변경 시마다 재실행해야 한다.

## 의존성

### 외부 패키지
- `fs` (Node.js 내장) — 파일 시스템 접근
- `path` (Node.js 내장) — 경로 조작

### 저장소 내부 모듈
없음. (출력 파일 `src/generated/examples-list.ts`는 이 스크립트가 생성하는 대상이며 import하지 않는다.)

## Exports

이 파일은 ES 모듈로 `export default`나 named export를 제공하지 않는다. 스크립트 최하단에서 `generateExamplesBundle()`을 직접 호출하므로, Node.js로 실행 시 즉시 동작한다.

## 상세 명세

### 상수

- `SPEC_EXAMPLES_DIR`: `import.meta.dirname`을 기준으로 `../../../../specification/v0_9/catalogs/basic/examples`를 resolve한 절대 경로. 예시 JSON 파일들이 위치하는 specification 디렉토리다.
- `OUT_FILE`: `import.meta.dirname`을 기준으로 `../src/generated/examples-list.ts`를 resolve한 절대 경로. 생성될 TypeScript 파일의 경로다.

### `generateExamplesBundle(): void`

매개변수 없음. 반환값 없음.

동작 단계:

1. `SPEC_EXAMPLES_DIR`의 존재 여부를 확인한다. 디렉토리가 없으면 오류 메시지를 출력하고 `process.exit(1)`로 종료한다.
2. 해당 디렉토리를 읽어 `.json`으로 끝나는 파일들만 필터링한 뒤 알파벳 순으로 정렬한다.
3. 각 파일에 대해:
   - `src/generated/examples-list.ts`에서 specification 예시 폴더로의 상대 경로(`../../../../../specification/v0_9/catalogs/basic/examples/${file}`)를 계산한다.
   - 변수명을 `example_0`, `example_1`, ... 형태로 생성한다.
   - import 구문과 객체 엔트리(`'파일명': { default: 변수명 }`)를 각각의 배열에 추가한다.
4. 생성될 TypeScript 파일의 내용을 문자열로 조립한다. 내용에는 다음이 포함된다:
   - 자동 생성 경고 주석
   - `@a2ui/web_core/v0_9`에서 `A2uiMessage`를 import하는 구문
   - 개별 JSON 파일을 import하는 구문들
   - `ExampleData` 인터페이스: `messages?: A2uiMessage[]`, `description?: string` 필드 선택적 포함
   - `ExampleModule` 인터페이스: `default: ExampleData | A2uiMessage[]` 필드 포함
   - `exampleModules` 상수: `Record<string, ExampleModule>` 타입으로 cast되며, 키는 파일명, 값은 `{ default: 변수명 }` 구조
5. 출력 디렉토리(`src/generated/`)가 없으면 `fs.mkdirSync`로 재귀적으로 생성한다.
6. `fs.writeFileSync`로 조립된 내용을 UTF-8 인코딩으로 `OUT_FILE`에 저장한다.
7. 성공 메시지를 콘솔에 출력한다.

## 동작 흐름

스크립트 파일이 Node.js로 실행되면 모듈 최하단의 `generateExamplesBundle()` 호출이 즉시 실행된다. 이 함수는 specification 디렉토리를 스캔하여 JSON 파일 목록을 수집하고, 각 파일에 대한 import 구문과 매핑 엔트리를 생성한 뒤, 두 인터페이스(`ExampleData`, `ExampleModule`)와 `exampleModules` 상수를 포함하는 TypeScript 파일을 `src/generated/examples-list.ts` 경로에 기록한다.
