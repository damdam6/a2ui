# renderers/angular/scripts/generate-examples.mjs

## 개요

A2UI 스펙 저장소의 예제 JSON 파일들을 읽어 Angular 탐색기 앱(`a2ui_explorer`)이 사용하는 TypeScript 번들 파일을 생성하는 Node.js 스크립트다. v0.8과 v0.9 두 스펙 버전의 카탈로그 예제를 병합하여 단일 `.ts` 파일로 출력하며, minimal 카탈로그의 `catalogId`를 basic으로 교체하는 옵션을 제공한다. CLI 인수로 출력 경로·카탈로그 이름·오버라이드 여부를 제어할 수 있다.

## 의존성

### 외부 패키지
- `fs` (Node.js 내장) — 파일 읽기/쓰기/존재 확인
- `path` (Node.js 내장) — 경로 조합
- `node:util` — `parseArgs` 함수

### 저장소 내부 모듈
없음 (독립 스크립트)

## Exports

이 파일은 ESM 모듈이지만 최상위 `export` 구문을 갖지 않는다. `main()` 함수를 스크립트 진입점으로 즉시 실행한다.

## 상세 명세

### 상수: `DEFAULT_OUT_FILE`
- 값: `'a2ui_explorer/src/app/generated/examples-bundle.ts'`
- 생성된 TypeScript 번들의 기본 출력 경로.

### 상수: `DEFAULT_CATALOGS`
- 값: `['minimal', 'basic']`
- 인수 미지정 시 처리할 기본 카탈로그 이름 목록.

### 상수: `options`
- `parseArgs`에 전달되는 옵션 스키마 객체.
- `help` (`boolean`, short: `'h'`): 도움말 출력 후 종료.
- `out-file` (`string`, short: `'o'`, 기본값: `DEFAULT_OUT_FILE`): 출력 파일 경로.
- `catalog` (`string`, short: `'c'`, `multiple: true`, 기본값: `DEFAULT_CATALOGS`): 처리할 카탈로그 이름 목록. 여러 번 지정 가능.
- `override-minimal-catalog-id` (`boolean`, 기본값: `true`): minimal 카탈로그의 `catalogId`를 basic으로 교체할지 여부. `--no-override-minimal-catalog-id`로 비활성화.

### 상수: `HELP_MESSAGE`
- 실행 방법과 모든 옵션을 설명하는 멀티라인 문자열. `--help` 플래그 시 출력.

### 함수: `overrideMessagesCatalogId(messages)`
- 매개변수: `messages` — 메시지 객체 배열 (타입 명시 없음)
- 반환 타입: `void`
- 동작: 배열을 순회하여 각 메시지에 `createSurface.catalogId` 속성이 있으면 경로 내 `'catalogs/minimal/catalog.json'`을 `'catalogs/basic/catalog.json'`으로 치환한다. v0.8 `beginRendering` 메시지는 `catalogId`를 사용하지 않으므로 처리 대상에서 제외됨을 주석으로 명시.
- 내부 헬퍼: `overrideCatalogId(catalogId: string): string` — 단순 `String.replace` 호출.

### 함수: `readExamples(specPath, catalogs, overrideCatalogId, version)`
- 매개변수:
  - `specPath: string` — 스펙 디렉토리 루트 경로
  - `catalogs: string[]` — 읽을 카탈로그 이름 목록
  - `overrideCatalogId: boolean` — minimal 카탈로그 오버라이드 여부
  - `version: string` — 예제에 태깅할 버전 문자열 (`'0.8'` 또는 `'0.9'`)
- 반환 타입: 예제 객체 배열
- 동작:
  1. 각 카탈로그에 대해 `specPath/<catalog>/examples` 디렉토리 존재 여부를 확인한다.
  2. 존재하면 `.json` 파일만 필터링해 알파벳 순 정렬 후 순회한다.
  3. 파일명에서 확장자 제거 → 선행 숫자+언더스코어 패턴 제거(`/^[0-9]+_/`) → 하이픈·언더스코어를 공백으로 치환 → 각 단어 첫 글자 대문자화하여 `nameFromFile`을 생성한다.
  4. 파싱된 JSON이 배열이면 직접 `messages`로 사용, 객체이면 spread하여 `messages`, `name`, `description` 필드를 병합한다.
  5. 버전이 `'0.8'`이면 이름 뒤에 `(${catalog})` 접미사를 붙인다.
  6. 카탈로그가 `'minimal'`이고 `overrideCatalogId`가 `true`이면 `overrideMessagesCatalogId`를 호출한다.
  7. JSON 파싱 실패 시 파일 경로를 포함한 오류 메시지로 `Error`를 던진다.

### 함수: `main()`
- 시그니처: `async function main(): Promise<void>`
- 동작:
  1. `parseArgs({options, allowNegative: true})`로 CLI 인수를 파싱한다.
  2. `--help`이면 `HELP_MESSAGE`를 출력하고 반환한다.
  3. 출력 디렉토리가 없으면 `fs.mkdirSync(..., {recursive: true})`로 생성한다.
  4. `readExamples`를 두 번 호출한다: v0.8 경로(`../../specification/v0_8/json/catalogs`), v0.9 경로(`../../specification/v0_9/catalogs`).
  5. 결과를 `JSON.stringify(..., null, 2)`로 직렬화하여 TypeScript 소스 문자열을 조립한다. 출력 파일은 `EXAMPLES_V08`, `EXAMPLES_V09`, `EXAMPLES`(v0.9를 기본값으로) 세 개의 `export const`를 포함한다.
  6. `fs.writeFileSync`로 파일을 기록하고 경로를 콘솔에 출력한다.
- 오류 처리: `main().catch(err => { console.error(err); process.exit(1); })` 패턴으로 비정상 종료.

## 동작 흐름

스크립트 실행 → CLI 인수 파싱 → (help이면 종료) → 출력 디렉토리 확인/생성 → v0.8 예제 읽기 → v0.9 예제 읽기 → TypeScript 소스 문자열 생성 → 파일 쓰기 → 완료 메시지 출력
