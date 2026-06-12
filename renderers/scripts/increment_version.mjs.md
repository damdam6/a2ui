# renderers/scripts/increment_version.mjs

## 개요

워크스페이스 내 특정 패키지의 버전을 올리는 CLI 스크립트다. 대상 패키지의 `package.json`을 직접 수정하고, 그 패키지에 내부 의존성을 갖는 모든 패키지 디렉토리에서 `yarn install`을 실행해 lockfile을 최신 상태로 동기화한다. `--skip-sync` 옵션으로 동기화 단계를 생략할 수 있으며, `--help`로 사용법을 출력할 수 있다.

## 의존성

### 외부 패키지
- `node:fs` — `readFileSync`, `writeFileSync`
- `node:util` — `parseArgs`

### 저장소 내부 모듈
- [`./lib/workspace.mjs`](./lib/workspace.mjs.md) — `getPackageGraph`, `incrementVersion`, `runCommand`

## Exports

이 파일은 모듈 export를 하지 않는다. `#!/usr/bin/env node` 선언이 있는 독립 실행 스크립트다.

## 상세 명세

### CLI 인수 파싱

`parseArgs`를 사용해 `process.argv.slice(2)`를 파싱한다. 지원하는 옵션은 다음과 같다.

- `--skip-sync` (boolean, 기본값 `false`): 의존 패키지의 `yarn install` 동기화를 건너뛴다.
- `--help` / `-h` (boolean, 기본값 `false`): 사용법 메시지를 출력하고 종료한다.
- positionals[0]: 대상 패키지 이름 (`targetName`)
- positionals[1]: 설정할 버전 문자열 (`targetVersion`, 선택)

`--help`가 지정된 경우 사용법·인수 설명·예제를 `console.log`로 출력한 뒤 `process.exit(0)`으로 종료한다.

`targetName`이 제공되지 않으면 오류 메시지를 출력하고 `process.exit(1)`으로 종료한다.

### 패키지 검색 로직

`getPackageGraph()`로 워크스페이스 전체 패키지 그래프를 얻는다. `graph[targetName]`으로 정확한 이름 매칭을 먼저 시도하고, 실패하면 `p.name.endsWith('/' + targetName) || p.name === targetName` 조건으로 접미사 매칭을 시도한다. 예를 들어 `'lit'`을 입력하면 `'@a2ui/lit'`에 매칭된다. 여전히 찾지 못하면 `console.error`로 에러를 출력하고 `process.exit(1)`로 종료한다.

### 버전 결정 및 `package.json` 수정

기존 버전은 `pkg.version`에서 읽는다. `targetVersion` 인수가 제공되었으면 그 값을 사용하고, 없으면 `incrementVersion(oldVersion)`의 반환값을 사용한다. 대상 패키지의 `package.json`을 `readFileSync`로 읽어 파싱한 뒤 `version` 필드를 교체하고, `JSON.stringify(pkgJson, null, 2) + '\n'` 형식으로 다시 저장한다.

### 의존 패키지 동기화

`graph`의 모든 패키지 중 `p.internalDependencies.includes(pkg.name)`인 것들을 `dependents`로 수집한다. `dependents`가 존재하고 `skipSync`가 `false`인 경우 각 의존 패키지를 순회한다.

- 패키지 이름이 `'@a2ui/custom-components-example'`인 경우 해당 패키지는 외부 의존성 문제로 건너뛴다 (skip 로그 출력).
- 그 외에는 해당 패키지 디렉토리(`dep.dir`)에서 `runCommand('yarn', ['install'], {cwd: dep.dir})`를 실행한다.

## 동작 흐름

1. CLI 인수 파싱 → `--help`이면 출력 후 종료, `targetName` 없으면 에러 후 종료.
2. `getPackageGraph()`로 전체 패키지 그래프 조회.
3. 정확한 이름 또는 접미사로 대상 패키지 탐색 → 없으면 에러 종료.
4. 새 버전 결정 (인수 우선, 없으면 자동 증가).
5. 대상 `package.json`의 `version` 필드를 덮어쓴다.
6. 내부 의존 패키지 목록을 필터링하고, `--skip-sync`가 없으면 각 패키지에서 `yarn install` 실행 (단, `@a2ui/custom-components-example` 제외).
7. `'Done.'` 출력 후 종료.
