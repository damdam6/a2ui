# renderers/react/scripts/postbuild.mjs

## 개요

빌드 후처리 스크립트로, TypeScript 컴파일러가 CSS 임포트 진입점(`styles/index.ts`)에 대해 생성하는 선언 파일을 `dist` 디렉터리에 복사한다. 루트 스타일과 `v0_8` 스타일 각각에 대해 `.d.ts`와 `.d.cts` 두 가지 형식으로 복제하여 CommonJS 및 ESM 환경 모두에서 타입이 올바르게 해석되도록 보장한다. `package.json`의 `postbuild` 훅에서 호출되며 Node.js에서 직접 실행된다.

## 의존성

### 외부 패키지
- `node:fs` — `copyFileSync`, `mkdirSync`, `existsSync`
- `node:path` — `join`

### 저장소 내부 모듈
없음. 이 스크립트는 순수 Node.js 파일 시스템 작업만 수행한다.

## Exports

없음. 실행 가능한 스크립트이며 모듈로서 내보내는 항목이 없다.

## 상세 명세

### 전역 상수

- `root`: `process.cwd()`로 현재 작업 디렉터리(저장소 내 `renderers/react/` 루트)를 나타낸다.
- `dist`: `join(root, 'dist')` — 빌드 결과물이 위치하는 디렉터리 경로.
- `src`: `join(root, 'src')` — 소스 파일이 위치하는 디렉터리 경로.

스크립트 시작 시 `dist` 디렉터리가 존재하지 않으면 `mkdirSync(dist, {recursive: true})`로 생성한다.

### `copyDts(source: string, destinationDir: string): void`

단일 `.d.ts` 파일 하나를 `destinationDir` 아래에 `index.d.ts`와 `index.d.cts` 두 이름으로 복사한다.

**동작 단계:**
1. `source` 인자를 `dts` 변수에 그대로 저장한다 (경로 재사용).
2. 목적지 경로 `targetDts = join(destinationDir, 'index.d.ts')`와 `targetDcts = join(destinationDir, 'index.d.cts')`를 계산한다.
3. `destinationDir`이 존재하지 않으면 `mkdirSync(destinationDir, {recursive: true})`로 생성한다.
4. `copyFileSync(dts, targetDts)`로 ESM 타입 선언 파일을 복사한다.
5. `copyFileSync(dts, targetDcts)`로 동일 파일을 CJS 타입 선언 파일로 복사한다.

### 스크립트 본체 실행 흐름

1. **루트 스타일 복사**: `copyDts(join(src, 'styles', 'index.d.ts'), join(dist, 'styles'))`를 호출하여 `src/styles/index.d.ts`를 `dist/styles/`에 배치한다.
2. **`v0_8` 스타일 디렉터리 생성**: `mkdirSync(join(dist, 'v0_8', 'styles'), {recursive: true})`로 중간 경로를 모두 포함해서 생성한다.
3. **`v0_8` 스타일 복사**: `copyDts(join(src, 'v0_8', 'styles', 'index.d.ts'), join(dist, 'v0_8', 'styles'))`로 `v0_8` 전용 선언 파일을 배치한다.
4. 성공 메시지 `'Post-build style declarations copied successfully.'`를 `console.log`로 출력하고 종료한다.

## 동작 흐름

스크립트는 위에서 아래로 순차 실행된다. 먼저 경로 상수를 초기화하고, `dist` 디렉터리 존재를 보장한 뒤, `copyDts` 함수를 두 번 호출하여 루트 및 버전별 스타일 선언 파일을 각각 복사한다. 중간 디렉터리는 `{recursive: true}` 옵션으로 자동 생성되므로 별도의 오류 처리 없이 진행된다.
