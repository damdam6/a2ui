# renderers/react/eslint.config.mjs

## 개요

`renderers/react` 패키지의 ESLint 설정을 ESM 모듈 형식(`.mjs`)으로 제공하는 파일이다. 저장소 루트의 공유 preset(`eslint.preset.mjs`)을 가져와 스프레드한 뒤, `visual-parity/**` 경로를 lint 대상에서 제외하는 단일 규칙을 추가한다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈
- `../../eslint.preset.mjs` — 저장소 루트의 공유 ESLint preset (상대 경로 `../../eslint.preset.mjs`, repo-root 기준 `eslint.preset.mjs`)

## Exports

기본(default) export: ESLint flat config 배열

## 상세 명세

### default export (배열)

`[...preset, {ignores: ['visual-parity/**']}]` 형태의 flat config 배열이다.

- `...preset`: 저장소 루트 공유 preset의 모든 규칙을 스프레드로 포함한다.
- `{ignores: ['visual-parity/**']}`: `visual-parity` 디렉터리 하위의 모든 파일을 lint에서 제외한다.

## 동작 흐름

이 파일은 `eslint.config.js`와 별도로 `.mjs` 확장자를 사용하는 ESM 전용 설정 파일이다. ESLint가 `.mjs` 형식의 설정을 탐색할 때 로드되며, 루트 preset을 최소한의 오버라이드로 재사용한다.
