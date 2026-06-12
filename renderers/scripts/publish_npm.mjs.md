# renderers/scripts/publish_npm.mjs

## 개요

워크스페이스 패키지를 npm(또는 Artifact Registry)에 배포하는 메인 스크립트다. 패키지 간 의존성 위상정렬, npm 버전 충돌 사전 검증, Git 상태 확인, 빌드/테스트 실행, 실제 publish 단계를 순서대로 수행한다. dry-run 모드를 기본값으로 채택해 `--no-dry-run` 옵션을 명시적으로 전달해야 실제 배포가 실행된다. `main` 함수를 export하므로 테스트에서 목업 의존성을 주입할 수 있다.

## 의존성

### 외부 패키지
- `node:child_process` — `execSync`
- `node:fs` — `readFileSync`
- `node:path` — `join`
- `node:url` — `fileURLToPath`
- `node:util` — `parseArgs`

### 저장소 내부 모듈
- [`./lib/workspace.mjs`](./lib/workspace.mjs.md) — `ansi`, `getPackageGraph`, `maybeRunCommand`, `runCommand`

## Exports

| 이름 | 종류 |
|---|---|
| `main` | 비동기 함수 (async function) |

## 상세 명세

### `topologicalSort(packageNames, graph)` → `string[]` (비공개)

- `packageNames` (string[]): 정렬할 패키지 이름 목록.
- `graph` (object): `getPackageGraph()`의 반환값.
- 반환값: 의존성 순서대로 정렬된 패키지 이름 배열 (의존하는 쪽이 뒤에 위치).

DFS 기반 위상정렬을 구현한다. 방문 완료 집합 `visited`와 순환 감지용 임시 집합 `temp`를 사용한다. `visit(name)` 내부 함수는 `temp`에 이름이 이미 있으면 순환 의존성 오류를 throw하고, `visited`에 있으면 즉시 반환한다. 그 외에는 `pkg.internalDependencies` 중 `packageNames`에 포함된 것을 재귀 방문한 뒤, `visited`에 추가하고 `sorted`에 push한다.

### `getVersionDiff(oldV, newV)` → `string` (비공개)

- `oldV`, `newV` (string): 비교할 버전 문자열.
- 반환값: `'SAME'` | `'MAJOR'` | `'MINOR'` | `'PATCH'` | `'PREMAJOR'` | `'PREMINOR'` | `'PREPATCH'` | `'PRERELEASE'` | `'GRADUATION (RELEASE)'` | `'OLDER_OR_UNKNOWN'`

두 버전을 `-`로 분리해 코어(`x.y.z`)와 pre-release 부분을 추출한다. 코어 부분을 숫자 3-tuple로 비교한다. 신규 버전에 pre-release 접미사가 있으면 `PRE*` 접두사를 붙인다. 코어는 동일하지만 old에는 pre-release가 있고 new에는 없으면 `'GRADUATION (RELEASE)'`. 반대 방향(일반 → pre)이거나 숫자가 줄어든 경우는 `'OLDER_OR_UNKNOWN'`을 반환한다.

### `checkGitProvenance(exec)` → `string` (비공개)

- `exec` (Function): `execSync`와 동일한 시그니처를 갖는 함수.
- 반환값: 현재 Git 커밋 해시 문자열.

`git rev-parse --abbrev-ref HEAD`, `git rev-parse HEAD`, `git status --porcelain`을 실행해 현재 브랜치, 커밋 해시, dirty 상태를 확인한다. `try/catch`로 감싸 실패 시 경고 메시지를 출력하고 계속 진행한다. `isDirty`가 true이면 uncommitted 변경사항 경고를 노란색으로 출력한다. 최종적으로 `commitHash`를 반환한다.

### `checkCoreDependencies(packageNames)` → `void` (비공개)

- `packageNames` (string[]): 배포할 패키지 이름 목록.

렌더러 목록 `['@a2ui/lit', '@a2ui/angular', '@a2ui/react']` 중 `packageNames`에 포함된 항목이 있을 때, `@a2ui/web_core`와 `@a2ui/markdown-it`도 함께 포함되어 있는지 검사한다. 하나라도 빠진 경우 경고 메시지를 출력하고 `Error`를 throw한다.

### `checkNpmVersions(packages, exec)` → `void` (비공개)

- `packages` (object[]): 패키지 정보 객체 배열.
- `exec` (Function): `execSync`와 동일한 시그니처.

각 패키지에 대해 `yarn npm info ${pkg.name} --fields version`을 실행해 npm에 등록된 최신 버전을 확인한다. 실패(네트워크 오류 등)하면 `remoteVersion`을 `null`로 처리해 신규 패키지로 간주한다.

- `remoteVersion`이 null: 신규 패키지로 출력하고 계속 진행.
- `remoteVersion === localVersion`: 버전이 이미 publish되었으므로 `Error` throw.
- `getVersionDiff(remoteVersion, localVersion) === 'OLDER_OR_UNKNOWN'`: 로컬 버전이 npm보다 낮거나 유효하지 않으므로 `Error` throw.
- 그 외: diff 유형과 함께 `✅` 로그 출력.

### `buildAndTestPackages(packages, runCmd, skipTests)` → `void` (비공개)

- `packages` (object[]): 패키지 정보 객체 배열.
- `runCmd` (Function): `runCommand`와 동일한 시그니처.
- `skipTests` (boolean): `true`이면 테스트 단계를 건너뜀.

각 패키지 디렉토리에서 `yarn install`을 실행한다. `skipTests`가 `false`이면 패키지의 `package.json`을 읽어 `scripts['test:ci']`가 존재하면 그것을, 없으면 `'test'`를 스크립트로 사용해 `yarn run <testScript>`를 실행한다.

### `main(args, mocks?)` → `Promise<void>` (export)

- `args` (string[]): 파싱할 CLI 인수 배열.
- `mocks` (object, 기본값 `{}`):
  - `mocks.runCommand` (Function): `runCommand` 대체 구현.
  - `mocks.execSync` (Function): `execSync` 대체 구현.

`parseArgs`로 다음 옵션을 파싱한다 (`allowNegative: true` 포함):
- `-p` / `--package` (string[], multiple, 기본값 `[]`): 배포할 패키지 이름.
- `--check-core-dependencies` (boolean, 기본값 `true`): 코어 의존성 검사 여부.
- `--dry-run` (boolean, 기본값 `true`): dry-run 모드 여부.
- `--skip-tests` (boolean, 기본값 `false`): 테스트 건너뜀 여부.
- `-h` / `--help` (boolean): 도움말 출력 후 조기 반환.

패키지 목록이 비어 있으면 `Error`를 throw한다.

실행 순서:
1. `checkGitProvenance(exec)`로 Git 상태 확인 및 커밋 해시 획득.
2. `getPackageGraph()`로 그래프 로드.
3. 짧은 이름을 전체 이름으로 resolve; 없으면 `Error` throw.
4. `checkCoreDeps`가 true이면 `checkCoreDependencies` 호출.
5. `topologicalSort`로 배포 순서 결정.
6. `checkNpmVersions`로 버전 충돌 검사.
7. `buildAndTestPackages`로 빌드 및 테스트.
8. 각 패키지에 대해 NPM 토큰 결정 후 `yarn run publish:package` 실행.
   - `dryRun`이 false이고 `process.env.NPM_TOKEN`이 없으면 `gcloud auth print-access-token`으로 토큰 획득.
   - `maybeRunCommand('yarn', ['run', 'publish:package'], {cwd, env: {...process.env, NPM_TOKEN}}, {dryRun, runCommand})`로 실행.

### 직접 실행 가드

`process.argv[1] === fileURLToPath(import.meta.url)` 조건으로 직접 실행 여부를 판별해, 직접 실행 시에만 `main(process.argv.slice(2))` 호출하고 오류는 빨간색으로 출력한 뒤 `process.exit(1)`.

## 동작 흐름

스크립트는 크게 검증 → 준비 → 배포 세 단계로 흐른다. Git 상태 확인 및 npm 버전 사전 검사로 잘못된 배포를 막고, 위상정렬로 의존성 순서를 보장한 뒤, `maybeRunCommand`를 통해 dry-run/실제 실행을 투명하게 전환한다.
