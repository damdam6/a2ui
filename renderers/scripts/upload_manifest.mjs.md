# renderers/scripts/upload_manifest.mjs

## 개요

npm 배포 매니페스트를 생성하고 Google Cloud Storage(GCS)에 업로드하는 스크립트다. 패키지 목록이 지정되면 해당 패키지만 포함하는 선택적 매니페스트를 생성하고, 지정되지 않으면 전체 배포(`publish_all: true`) 매니페스트를 생성한다. 생성된 JSON 파일을 저장소 루트에 `manifest.json`으로 저장한 뒤 `gcloud storage cp` 명령으로 GCS에 업로드한다. dry-run 모드를 기본값으로 사용하며 `main` 함수를 export해 테스트에서 목업 주입이 가능하다.

## 의존성

### 외부 패키지
- `node:fs` — `writeFileSync`
- `node:path` — `join`
- `node:util` — `parseArgs`
- `node:url` — `fileURLToPath`

### 저장소 내부 모듈
- [`./lib/workspace.mjs`](./lib/workspace.mjs.md) — `getPackageGraph`, `ROOT_DIR`, `ansi`, `maybeRunCommand`

## Exports

| 이름 | 종류 |
|---|---|
| `main` | 비동기 함수 (async function) |

## 상세 명세

### `GCS_URI` (모듈 수준 상수, string)

`process.env.A2UI_NPM_MANIFEST_GCS_URI` 환경변수 값을 사용하거나, 미설정 시 기본값 `'gs://oss-exit-gate-prod-projects-bucket/a2ui/npm/manifests'`를 사용한다.

### `main(args, mocks?)` → `Promise<void>` (export)

- `args` (string[]): 파싱할 CLI 인수 배열.
- `mocks` (object, 기본값 `{}`):
  - `mocks.runCommand` (Function | undefined): `maybeRunCommand`의 `settings.runCommand`로 전달되는 목업.
  - `mocks.writeFileSync` (Function): `writeFileSync` 대체 구현 (기본값: 실제 `writeFileSync`).

`parseArgs`로 다음 옵션을 파싱한다 (`allowNegative: true` 포함):
- `-p` / `--package` (string[], multiple, 기본값 `[]`): 배포할 패키지 이름.
- `--dry-run` (boolean, 기본값 `true`): dry-run 모드 여부.

#### 매니페스트 생성 로직

`packagesToPublish.length > 0`이면 선택적 매니페스트를 생성한다.
- 각 패키지 이름을 `graph` 에서 탐색할 때 `p.dir.includes('/renderers/')`인 패키지만 대상으로 하고, `p.name === name || p.name.endsWith('/' + name)` 조건으로 매칭한다. 없으면 `Error` throw.
- 매칭된 패키지 이름에서 `'@a2ui/'` 접두사를 제거(`replace('@a2ui/', '')`)해 `resolvedPackages` 배열을 만든다.
- 매니페스트 구조:
  ```
  {
    publish_all: false,
    publishing_groups: [{ namespace: '@a2ui', packages: [{name}, ...] }]
  }
  ```

패키지 목록이 없으면 `{ publish_all: true }` 매니페스트를 생성한다.

#### 파일 저장

`manifestPath`는 `join(ROOT_DIR, 'manifest.json')`이다. `writeFile(manifestPath, JSON.stringify(manifest, null, 2) + '\n')`로 저장한다.

#### GCS 업로드

파일명 패턴: `manifest-${mainVersion}-${timestamp}.json`. `timestamp`는 `new Date().toISOString()`에서 `[:.]/g` 문자를 `'-'`로 치환하고 `slice(0, 19)`으로 잘라 `'YYYY-MM-DDTHH-mm-ss'` 형식으로 만든다. `mainVersion`은 `graph['@a2ui/web_core']?.version`에서 가져오며, `undefined`이면 `Error`를 throw한다.

`maybeRunCommand('gcloud', ['storage', 'cp', manifestPath, `${GCS_URI}/${manifestFileName}`], {}, {dryRun: isDryRun, runCommand: runCmd})`를 호출한다. 실패 시 `'Failed to upload manifest...'` 메시지와 함께 원인(`cause`)을 포함한 `Error`를 throw한다.

### 직접 실행 가드

`process.argv[1] === fileURLToPath(import.meta.url)` 조건으로 직접 실행 시에만 `main(process.argv.slice(2))` 호출하고 오류를 빨간색으로 출력한 뒤 `process.exit(1)`.

## 동작 흐름

1. CLI 인수 파싱 → 패키지 목록과 dry-run 플래그 결정.
2. `getPackageGraph()`로 워크스페이스 그래프 로드.
3. 패키지 목록 유무에 따라 매니페스트 내용 결정 (선택적 vs 전체).
4. `manifest.json`을 저장소 루트에 저장.
5. `@a2ui/web_core` 버전과 현재 타임스탬프로 GCS 업로드 파일명 결정.
6. `maybeRunCommand`로 dry-run 여부에 따라 조건부 업로드 실행.
