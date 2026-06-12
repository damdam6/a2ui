# renderers/scripts/publish_npm.test.mjs

## 개요

`publish_npm.mjs`의 `main` 함수를 대상으로 하는 통합 테스트 파일이다. Node.js 내장 `node:test` 모듈을 사용하며, `runCommand`와 `execSync`를 목업으로 주입해 실제 셸 명령 실행 없이 스크립트 전체 흐름을 검증한다.

## 의존성

### 외부 패키지
- `node:test` — `describe`, `it`
- `node:assert` — 기본 export

### 저장소 내부 모듈
- [`./publish_npm.mjs`](./publish_npm.mjs.md) — `main`

## Exports

없음. 테스트 파일은 직접 실행 전용이다.

## 테스트 케이스

### `describe('publish_npm script integration test')`

#### `'should topologically sort packages based on dependencies'`

**검증 동작**: `lit`, `web_core`, `markdown-it`을 입력 순서에 관계없이 제공했을 때 실행 명령 목록에서 `web_core`의 `install` 인덱스와 `markdown-it`의 `install` 인덱스가 모두 `lit`의 `install` 인덱스보다 앞에 위치하는지 확인한다.

**픽스처/모킹**:
- `runCommand`: 실행된 명령을 `"cmd args (in dirName)"` 형식으로 `executedCommands` 배열에 누적.
- `execSync`: `'npm info'` 포함 시 `'0.0.1\n'` 반환, 그 외 빈 문자열.
- 인수: `['--package=lit', '--package=web_core', '--package=markdown-it', '--skip-tests', '--no-dry-run']`

**검증**: `assert.ok(webCoreInstallIndex < litInstallIndex)`, `assert.ok(markdownItInstallIndex < litInstallIndex)`

---

#### `'should default to dry-run mode (skip auth and publish)'`

**검증 동작**: 기본 dry-run 모드에서는 `gcloud auth` 호출이 발생하지 않고 `publish:package` 명령이 실행되지 않아야 하며, `yarn install`은 실행되어야 한다.

**픽스처/모킹**:
- `runCommand`: `executedCommands` 배열에 누적.
- `execSync`: `'gcloud auth'` 포함 시 `gcloudCalled = true` 설정, `'npm info'` 포함 시 `'0.0.1\n'`, 그 외 빈 문자열.
- 인수: `['--package=web_core']` (dry-run이 기본값)

**검증**: `gcloudCalled === false`, `hasPublish === false`, `hasInstall === true`

---

#### `'should authenticate and publish when --no-dry-run is passed'`

**검증 동작**: `--no-dry-run` 전달 시 `gcloud auth` 토큰 취득이 발생하고 `publish:package` 명령이 실행된다.

**픽스처/모킹**:
- `runCommand`: `executedCommands` 배열에 누적.
- `execSync`: `'gcloud auth'` 포함 시 `gcloudCalled = true` 및 `'dummy_token\n'` 반환, `'npm info'` 포함 시 `'0.0.1\n'`, 그 외 빈 문자열.
- 인수: `['--package=web_core', '--no-dry-run']`

**검증**: `gcloudCalled === true`, `hasPublish === true`

---

#### `'should skip tests when --skip-tests is passed'`

**검증 동작**: `--skip-tests` 전달 시 `'run test'`가 포함된 명령이 실행되지 않아야 한다.

**픽스처/모킹**:
- `runCommand`: `executedCommands` 배열에 누적.
- `execSync`: `'npm info'` 포함 시 `'0.0.1\n'`, 그 외 빈 문자열.
- 인수: `['--package=web_core', '--skip-tests']`

**검증**: `hasTest === false`

---

#### `'should fail safety check if core dependencies are missing'`

**검증 동작**: 코어 패키지(`web_core`, `markdown-it`) 없이 `lit`만 배포하려 할 때 `main`이 `Error`를 reject해야 한다.

**픽스처/모킹**:
- `runCommand`: no-op.
- `execSync`: 빈 문자열 반환.
- 인수: `['--package=lit']`

**검증**: `assert.rejects(main(...), /.*/)`로 오류 발생 확인.

---

#### `'should bypass safety check when --no-check-core-dependencies is passed'`

**검증 동작**: `--no-check-core-dependencies` 전달 시 코어 패키지 없이도 `lit` 배포가 진행되어 `yarn install`이 실행된다.

**픽스처/모킹**:
- `runCommand`: `executedCommands` 배열에 누적.
- `execSync`: `'npm info'` 포함 시 `'0.0.1\n'`, 그 외 빈 문자열.
- 인수: `['--package=lit', '--no-check-core-dependencies']`

**검증**: `hasInstall === true`

---

#### `'should output help message and return early when --help is passed'`

**검증 동작**: `--help` 전달 시 어떤 명령도 실행하지 않고 `main`이 조기 반환된다.

**픽스처/모킹**:
- `runCommand`: `executedCommands` 배열에 누적.
- `execSync`: 빈 문자열 반환.
- 인수: `['--help']`

**검증**: `executedCommands.length === 0`
