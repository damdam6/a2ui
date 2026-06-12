# renderers/markdown/markdown-it/eslint.config.mjs

## 개요

이 파일은 `markdown-it` 렌더러 패키지의 ESLint 설정 파일이다. 저장소 루트에 위치한 공유 ESLint 프리셋(`eslint.preset.mjs`)을 가져와 그대로 재노출함으로써, 패키지 전역 린팅 규칙을 루트 설정과 일관되게 유지한다. 별도의 규칙을 추가하거나 오버라이드하지 않는다.

## 의존성

### 저장소 내부 모듈

- [`../../../eslint.preset.mjs`](../../../eslint.preset.mjs.md) — 저장소 루트의 공유 ESLint 프리셋. 스프레드(`...preset`)로 그대로 확장된다.

### 외부 패키지

없음.

## Exports

- `default` (배열) — `preset` 배열을 스프레드한 결과를 기본 내보내기로 제공한다. ESLint Flat Config 형식을 따른다.

## 상세 명세

### default export

`preset`(배열)을 `[...preset]`으로 스프레드하여 새 배열을 만든 뒤 `export default`로 내보낸다. 이 구조는 ESLint Flat Config 규격을 준수하며, 향후 배열에 추가 규칙 객체를 덧붙일 수 있는 확장 지점 역할을 한다. 현재는 프리셋을 수정 없이 그대로 전달한다.

## 동작 흐름

1. 루트 `eslint.preset.mjs`를 `preset`으로 import한다.
2. `[...preset]`을 default export로 노출한다.
3. ESLint가 이 파일을 읽어 `markdown-it` 패키지 디렉토리 전체의 린팅에 적용한다.
