# renderers/web_core/scripts/copy-spec.js

## 개요

`specification` 디렉토리에 있는 JSON 스키마 파일과 카탈로그 파일을 `web_core` 패키지의 `src/<version>/schemas/` 경로로 복사하는 빌드 스크립트다. Windows/Unix 크로스 플랫폼 호환성을 위해 셸 명령어 대신 Node.js 내장 `fs`/`path` 모듈만 사용한다. 현재 `v0_8`, `v0_9`, `v1_0` 세 버전에 대해 순차적으로 실행된다.

## 의존성

### 외부 패키지
- `node:fs` — `mkdirSync`, `cpSync`, `readdirSync`, `existsSync`
- `node:path` — `join`, `dirname`
- `node:url` — `fileURLToPath`

### 저장소 내부 모듈
없음 (순수 Node.js 스크립트)

## Exports

없음 (직접 실행 스크립트이며 모듈 내보내기 없음)

## 상세 명세

### `__dirname` (상수)
- **값:** `dirname(fileURLToPath(import.meta.url))`
- ESM 환경에서 `__dirname`이 기본 제공되지 않으므로 `import.meta.url`을 파일 시스템 경로로 변환한 뒤 디렉토리 부분을 추출해 `__dirname`을 직접 정의한다.

### `rootDir` (상수)
- **값:** `join(__dirname, '..')` — `scripts/` 의 상위 디렉토리, 즉 `web_core/` 루트

### `copySchemas(version: string): void`
- **매개변수:** `version` — 복사할 버전 문자열 (예: `'v0_8'`, `'v0_9'`, `'v1_0'`)
- **반환:** 없음

**동작 로직:**
1. 소스 JSON 디렉토리 경로 `srcJsonDir`을 `<rootDir>/../../specification/<version>/json`으로 계산한다.
2. 소스 카탈로그 디렉토리 경로 `srcCatalogsDir`을 `<rootDir>/../../specification/<version>/catalogs`로 계산한다.
3. 대상 디렉토리 `destDir`을 `<rootDir>/src/<version>/schemas`로 계산한다.
4. `mkdirSync(destDir, {recursive: true})`로 대상 디렉토리가 없으면 생성한다.
5. `srcJsonDir`이 존재하면(`existsSync`), 디렉토리 내 파일을 읽어 `.json`으로 끝나는 파일만 필터링하고 각 파일을 `destDir`로 복사한다.
6. `version`이 `'v0_8'`이 **아닌** 경우에만 `srcCatalogsDir` 전체를 `destDir/catalogs`로 재귀 복사한다. `v0_8`은 카탈로그 파일을 복사하지 않는다.

### 진입점 실행
파일 하단에서 `copySchemas('v0_8')`, `copySchemas('v0_9')`, `copySchemas('v1_0')`를 순서대로 호출하여 세 버전 모두의 스키마를 복사한다.

## 동작 흐름

스크립트가 직접 실행되면 세 버전에 대해 `copySchemas`가 순차 호출된다. 각 호출에서 JSON 스키마 파일이 복사되고, `v0_8`을 제외한 버전에서는 카탈로그 디렉토리도 함께 복사된다. 대상 디렉토리는 없으면 자동 생성되므로 별도의 사전 준비 없이 실행 가능하다.
