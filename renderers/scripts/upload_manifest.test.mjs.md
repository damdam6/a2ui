# renderers/scripts/upload_manifest.test.mjs

## 개요

`upload_manifest.mjs`의 `main` 함수를 대상으로 하는 통합 테스트 파일이다. Node.js 내장 `node:test` 모듈을 사용하며, `runCommand`와 `writeFileSync`를 목업으로 주입해 실제 파일시스템 접근과 셸 명령 실행 없이 매니페스트 생성 및 업로드 흐름을 검증한다.

## 의존성

### 외부 패키지
- `node:test` — `describe`, `it`
- `node:assert` — 기본 export

### 저장소 내부 모듈
- [`./upload_manifest.mjs`](./upload_manifest.mjs.md) — `main`

## Exports

없음. 테스트 파일은 직접 실행 전용이다.

## 테스트 케이스

### `describe('upload_manifest script integration test')`

#### `'should generate manifest with publish_all: true by default'`

**검증 동작**: 패키지를 지정하지 않고 `main([])`을 호출했을 때 `manifest.json`에 `publish_all: true`가 기록되어야 한다.

**픽스처/모킹**:
- `runCommand`: no-op.
- `writeFileSync`: 두 번째 인수(`content`)를 `writtenFileContent`에 저장.
- 인수: `[]`

**검증**: `writtenFileContent`가 truthy, `JSON.parse(writtenFileContent).publish_all === true`

---

#### `'should skip upload in dry-run mode by default'`

**검증 동작**: 기본 dry-run 모드에서는 `runCommand`(gcloud 업로드)가 전혀 호출되지 않아야 한다.

**픽스처/모킹**:
- `runCommand`: `executedCommands` 배열에 누적.
- `writeFileSync`: no-op.
- 인수: `[]`

**검증**: `executedCommands.length === 0`

---

#### `'should generate manifest with publish_all: false when packages are specified'`

**검증 동작**: `--package=angular`, `-p lit`, `--package react` 세 패키지를 지정했을 때 `publish_all: false`이고 `publishing_groups`의 구조와 패키지 이름 순서가 정확해야 한다.

**픽스처/모킹**:
- `runCommand`: no-op.
- `writeFileSync`: `content`를 `writtenFileContent`에 저장.
- 인수: `['--package=angular', '-p', 'lit', '--package', 'react']`

**검증**:
- `manifest.publish_all === false`
- `manifest.publishing_groups` 존재, 길이 1
- `manifest.publishing_groups[0].namespace === '@a2ui'`
- `packages` 배열 길이 3, 순서대로 `'angular'`, `'lit'`, `'react'`

---

#### `'should attempt to upload to gcloud when --no-dry-run is passed'`

**검증 동작**: `--no-dry-run` 전달 시 `gcloud storage cp`로 시작하는 명령이 정확히 1회 실행되어야 한다.

**픽스처/모킹**:
- `runCommand`: `executedCommands` 배열에 누적.
- `writeFileSync`: no-op.
- 인수: `['--no-dry-run']`

**검증**: `executedCommands.length === 1`, `executedCommands[0].startsWith('gcloud storage cp') === true`
