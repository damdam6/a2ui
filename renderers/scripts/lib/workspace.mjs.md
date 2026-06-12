# renderers/scripts/lib/workspace.mjs

## 개요

워크스페이스 관리를 위한 공통 유틸리티 모듈로, 다른 스크립트들이 공유하는 핵심 기능을 제공한다. 저장소 루트에서 모든 `package.json`을 재귀 탐색해 패키지 그래프(이름, 버전, 내부 의존성)를 구축하고, 버전 문자열 자동 증가, 셸 명령 실행, dry-run 지원 명령 실행 등의 기능을 export한다.

## 의존성

### 외부 패키지
- `node:fs` — `readFileSync`, `readdirSync`, `statSync`, `existsSync`
- `node:path` — `join`, `resolve`, `relative`, `dirname`
- `node:child_process` — `spawnSync`
- `node:url` — `fileURLToPath`

### 저장소 내부 모듈
없음.

## Exports

| 이름 | 종류 |
|---|---|
| `ROOT_DIR` | 상수 (string) |
| `ansi` | 상수 (object) |
| `findPackages` | 함수 |
| `getPackageGraph` | 함수 |
| `incrementVersion` | 함수 |
| `runCommand` | 함수 |
| `maybeRunCommand` | 함수 |

## 상세 명세

### `ROOT_DIR` (상수, string)

`dirname(fileURLToPath(import.meta.url))`로 이 파일의 절대 디렉토리를 구한 뒤 `resolve(__dirname, '../../../')`로 저장소 루트 경로를 도출한다. `lib/workspace.mjs`는 `renderers/scripts/lib/` 안에 있으므로, 세 단계 위가 저장소 루트가 된다.

### `ansi` (상수, object)

터미널 출력 색상을 위한 ANSI 이스케이프 코드 모음이다.

- `yellow`: `'\x1b[33m'`
- `red`: `'\x1b[31m'`
- `green`: `'\x1b[32m'`
- `reset`: `'\x1b[0m'`

### `findPackages(dir?, packageList?)` → `string[]`

- `dir` (string, 기본값 `ROOT_DIR`): 탐색 시작 디렉토리.
- `packageList` (string[], 기본값 `[]`): 누적 결과 배열 (재귀 호출에 사용).
- 반환값: 찾은 `package.json`의 절대 경로 배열.

`readdirSync`로 `dir`의 항목을 열거한다. `'node_modules'`, `'.git'`, `'dist'`는 무시한다. 항목이 디렉토리이면 재귀 호출하고, 파일명이 `'package.json'`이면 절대 경로를 `packageList`에 추가한다. 최종적으로 `packageList`를 반환한다.

### `getPackageGraph()` → `Object`

반환값: 패키지 이름을 키로, 패키지 정보 객체를 값으로 갖는 맵.

패키지 정보 객체의 구조:
- `name` (string): `package.json`의 `name` 필드.
- `version` (string): `package.json`의 `version` 필드.
- `path` (string): `package.json`의 절대 경로.
- `dir` (string): `package.json`이 위치한 디렉토리 절대 경로.
- `dependencies`, `devDependencies`, `peerDependencies` (object): 각각의 의존성 맵 (없으면 `{}`).
- `private` (boolean): `!!pkg.private`.
- `internalDependencies` (string[]): 워크스페이스 내 다른 패키지 이름만 포함한 의존성 목록.

동작 단계:
1. `findPackages()`로 모든 `package.json` 경로를 수집한다.
2. 각 파일을 파싱해 `packages` 맵에 등록한다. 이름 충돌 시 우선순위 규칙을 적용한다.
   - 신규 패키지가 `'/renderers/'` 경로에 있고 기존 패키지가 그렇지 않으면 신규 우선.
   - 같은 위치(모두 renderers 이거나 모두 아님)이면 `scripts` 필드 수가 더 많은 쪽을 유지하되, 신규 패키지의 수가 같거나 적으면 `continue`(기존 유지).
3. 모든 패키지 등록 후 2차 패스에서 `internalDependencies`를 채운다. 각 패키지의 `dependencies + devDependencies + peerDependencies`를 합산해, 키(depName)가 `packages` 맵에 존재하는 것만 `internalDependencies`에 추가한다.

### `incrementVersion(version)` → `string`

- `version` (string): 증가시킬 버전 문자열.
- 반환값: 마지막 숫자 세그먼트를 1 증가시킨 버전 문자열.

버전 문자열을 `'.'`으로 분리해 마지막 부분을 정규식 `/^(.*?)(\d+)$/`로 매칭한다. 매칭 성공 시 숫자 부분을 `+1`하고 앞의 비숫자 접두사는 그대로 유지한다. 예시: `'0.9.2'` → `'0.9.3'`, `'0.9.2-beta.1'` → `'0.9.2-beta.2'`. 매칭 실패 시(숫자로 끝나지 않는 경우) `version + '.1'`을 반환한다.

### `commandToString(command, args, options?)` → `string` (비공개 헬퍼)

- `command` (string), `args` (string[]), `options` (object, 기본값 `{}`).
- 반환값: `"command arg1 arg2 [in /path]"` 형식의 사람이 읽을 수 있는 명령 문자열.
- `options.cwd`가 있으면 ` in ${options.cwd}` 를 뒤에 붙인다.

### `runCommand(command, args, options?)` → `SpawnSyncReturns`

- `command` (string), `args` (string[]), `options` (object, 기본값 `{}`).
- 반환값: `spawnSync`의 결과 객체.

`commandToString`으로 로그 문자열을 만들어 `console.log`로 출력한 뒤, `spawnSync(command, args, {stdio: 'inherit', shell: true, ...options})`를 실행한다. `result.status !== 0`이면 종료 코드와 명령 문자열을 포함한 `Error`를 throw한다.

### `maybeRunCommand(command, args, options?, settings?)` → `void`

- `command` (string), `args` (string[]), `options` (object, 기본값 `{}`).
- `settings` (object, 기본값 `{}`):
  - `settings.dryRun` (boolean, 기본값 `false`): dry-run 모드 여부.
  - `settings.runCommand` (Function, 선택): `runCommand`의 대체 구현 (테스트용 목업 지원).

`dryRun`이 `true`이면 `[DRY RUN] Did NOT execute:` 접두사와 함께 실행될 명령을 노란색으로 출력하고 실제로는 실행하지 않는다. `false`이면 `settings.runCommand` 또는 기본 `runCommand`를 호출한다.

## 동작 흐름

이 모듈은 직접 실행되지 않고 다른 스크립트에서 import된다. `ROOT_DIR`은 모듈 로드 시 즉시 계산되며, `ansi` 객체도 모듈 수준에서 초기화된다. 나머지 export는 모두 순수 함수 또는 유틸리티로 호출 시점에만 동작한다.
