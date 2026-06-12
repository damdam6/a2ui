# renderers/scripts/prepare-publish.mjs

## 개요

npm 배포를 위해 패키지를 준비하는 스크립트다. 소스 `package.json`을 읽어 내부 의존성 버전을 `file:`/`workspace:` 참조에서 semver 범위(`^x.y.z`)로 변환하고, `./dist/` 경로 접두사를 제거하여 배포 디렉토리에 정제된 `package.json`을 출력한다. 또한 `README.md`, `CHANGELOG.md`, 저장소 루트 `LICENSE` 파일을 배포 디렉토리로 복사한다.

## 의존성

### 외부 패키지
- `node:fs` — `readFileSync`, `writeFileSync`, `copyFileSync`, `existsSync`, `mkdirSync`
- `node:path` — `join`, `resolve`
- `node:url` — `fileURLToPath`

### 저장소 내부 모듈
- [`./lib/workspace.mjs`](./lib/workspace.mjs.md) — `getPackageGraph`

## Exports

이 파일은 모듈 export를 하지 않는다. 독립 실행 스크립트다 (하지만 shebang 라인은 없음).

## 상세 명세

### CLI 인수 파싱

`process.argv.slice(2)`를 순차적으로 순회하며 다음 옵션을 파싱한다.

- `--source <path>`: 소스 `package.json` 경로 (기본값 `'./package.json'`).
- `--dist <path>`: 출력 디렉토리 경로 (기본값 `'./dist'`).
- `--skip-path-adjustment`: `./dist/` 경로 접두사 제거를 건너뛴다 (boolean 플래그, 기본값 `false`).

`parseArgs`가 아닌 직접 구현한 루프를 사용하며, `args[i] === '--source'` 이면 `++i`로 다음 인수를 값으로 가져온다.

### 디렉토리 경로 계산

- `packageDir`: `process.cwd()` (스크립트를 실행하는 패키지의 루트).
- `scriptDir`: `fileURLToPath(new URL('.', import.meta.url))` (이 스크립트 파일의 디렉토리).
- `resolvedSourcePkg`: `resolve(packageDir, sourcePkgPath)`.
- `resolvedDistDir`: `resolve(packageDir, distDir)`.
- `rootDir`: `resolve(scriptDir, '../../')` (저장소 루트).

`resolvedDistDir`이 존재하지 않으면 `mkdirSync(resolvedDistDir, {recursive: true})`로 생성한다.

### `updateInternalDeps(deps)` (내부 함수)

- `deps` (object | undefined): `pkg.dependencies` 또는 `pkg.peerDependencies`.
- `deps`가 falsy이면 즉시 반환한다.
- 각 키(패키지 이름)를 순회해, 값이 `'file:'` 또는 `'workspace:'`로 시작하고 `graph[name]`이 존재하면 해당 값을 `'^' + graph[name].version` 으로 교체한다.

이 함수는 `pkg.dependencies`와 `pkg.peerDependencies`에 대해 순서대로 호출된다.

### 경로 조정 (path adjustment)

`--skip-path-adjustment`가 지정되지 않은 경우 실행된다.

내부 헬퍼 `adjustPath(p)`: `p`가 string이고 `'./dist/'`로 시작하면 `'./' + p.substring(7)`을 반환해 `./dist/` 접두사를 제거한다. 그 외에는 `p`를 그대로 반환한다.

조정 대상 필드:
- `pkg.main`, `pkg.module`, `pkg.types` — 단순 string 값에 `adjustPath` 적용.
- `pkg.exports` — 각 export 키를 순회하며 다음 구조를 처리한다.
  - string이면 직접 `adjustPath` 적용.
  - object이면 `types`, `default`, `import`, `require` 필드를 재귀적으로 처리. `import`와 `require`는 다시 string이거나 `{types, default}` 형태의 object일 수 있다.

### `package.json` 정제 및 저장

다음 필드를 `delete`로 제거한다: `scripts`, `wireit`, `files`, `prepublishOnly`, `devDependencies`.

정제된 객체를 `JSON.stringify(pkg, null, 2)` 형식으로 `join(resolvedDistDir, 'package.json')`에 저장한다 (trailing newline 없음).

### 파일 복사

- `join(packageDir, 'README.md')`가 존재하면 `resolvedDistDir/README.md`로 복사.
- `join(packageDir, 'CHANGELOG.md')`가 존재하면 `resolvedDistDir/CHANGELOG.md`로 복사.
- `join(rootDir, 'LICENSE')`가 존재하면 `resolvedDistDir/LICENSE`로 복사.

## 동작 흐름

1. CLI 인수 파싱으로 `sourcePkgPath`, `distDir`, `skipPathAdjustment` 결정.
2. 경로 계산 및 `dist` 디렉토리 보장.
3. `getPackageGraph()`로 워크스페이스 그래프 로드.
4. 소스 `package.json` 파싱.
5. 내부 의존성 버전 정규화 (`updateInternalDeps`).
6. 경로 접두사 제거 (`adjustPath`) — `skipPathAdjustment`가 false인 경우.
7. 개발 전용 필드 삭제.
8. 정제된 `package.json`을 `dist/`에 저장.
9. `README.md`, `CHANGELOG.md`, `LICENSE` 조건부 복사.
10. 완료 메시지 출력.
