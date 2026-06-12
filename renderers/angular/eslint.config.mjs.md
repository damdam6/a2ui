# renderers/angular/eslint.config.mjs

## 개요

`renderers/angular` 패키지의 ESLint 설정 파일이다. 저장소 루트의 공유 ESLint preset(`eslint.preset.mjs`)을 그대로 스프레드하여 사용하며, 이 패키지에 별도의 규칙 추가 없이 공통 설정을 상속한다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈
- `../../eslint.preset.mjs` — 저장소 루트의 공유 ESLint 설정 preset (repo-root 기준: `eslint.preset.mjs`)

## Exports

| 이름 | 종류 | 설명 |
|---|---|---|
| default | 배열 (ESLint flat config) | `[...preset]`으로 spread된 flat config 배열 |

## 동작 흐름

`preset`을 default import한 뒤 스프레드 연산자(`...`)로 배열을 그대로 펼쳐 default export한다. ESLint flat config 형식을 따르며, 별도 규칙 오버라이드 없이 루트 preset 설정을 완전히 위임한다. 모든 실제 린팅 규칙은 `../../eslint.preset.mjs`에 정의되어 있다.
