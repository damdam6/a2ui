# renderers/lit/eslint.config.mjs

## 개요

`renderers/lit` 패키지의 ESLint 설정 파일이다. 상위 공유 프리셋(`eslint.preset.mjs`)을 그대로 확장하면서, `a2ui_explorer/**` 경로 아래의 파일은 린트 검사에서 제외하는 단 하나의 추가 규칙을 적용한다.

## 의존성

### 저장소 내부 모듈
- [`../../eslint.preset.mjs`](../../eslint.preset.mjs.md) — 저장소 공통 ESLint 규칙 프리셋 (배열 형태로 export됨)

### 외부 패키지
없음

## Exports

- `default` (배열): 스프레드된 `preset` 배열에 `{ignores: ['a2ui_explorer/**']}` 객체를 추가한 flat config 배열

## 상세 명세

### `default` 설정 배열

1. `...preset` — 공유 프리셋의 모든 규칙 설정을 그대로 펼쳐서 포함한다.
2. `{ignores: ['a2ui_explorer/**']}` — `a2ui_explorer` 디렉토리 전체를 ESLint 검사 대상에서 제외한다. 이 서브 패키지는 별도 빌드 및 테스트 환경을 가지므로 렌더러 자체의 린트 규칙 적용 대상에서 분리한다.

## 동작 흐름

ESLint가 `renderers/lit/` 디렉토리에서 실행될 때 이 파일을 flat config로 로드한다. 공유 프리셋이 먼저 적용되고, 이후 `a2ui_explorer/**` 글로브 패턴에 해당하는 파일은 검사를 건너뛴다.
