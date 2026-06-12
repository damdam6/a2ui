# renderers/react/a2ui_explorer/eslint.config.js

## 개요

`a2ui_explorer` 하위 패키지 전용 ESLint 설정 파일이다. 상위 `renderers/react/eslint.config.js`의 설정을 스프레드 연산자로 전부 상속한 뒤, TypeScript 파서 옵션만 추가로 지정한다. `src/generated/**`, `dist/**`, `node_modules/**` 세 경로를 린트 대상에서 제외한다.

## 의존성

### 외부 패키지
없음 (Node.js 네이티브 `import.meta.dirname` 사용)

### 저장소 내부 모듈
- [`../eslint.config.js`](../eslint.config.js.md) — 부모 ESLint 설정을 `parentConfig`로 import

## Exports

기본(default) export: ESLint flat config 배열

## 상세 명세

### default export (배열)

두 개의 설정 객체를 담은 배열을 내보낸다.

1. **부모 설정 확장**: `...parentConfig` 스프레드로 `renderers/react/eslint.config.js`의 모든 규칙을 그대로 포함한다.
2. **TypeScript 파서 옵션 확장**:
   - `languageOptions.parserOptions.project`: `['./tsconfig.app.json', './tsconfig.spec.json', './tsconfig.node.json']` 세 tsconfig 파일을 지정한다.
   - `languageOptions.parserOptions.tsconfigRootDir`: `import.meta.dirname`을 사용하여 이 파일이 위치한 디렉터리를 루트로 설정한다.
3. **무시 경로 설정**:
   - `ignores`: `['src/generated/**', 'dist/**', 'node_modules/**']`

## 동작 흐름

모듈이 로드되면 부모 설정 배열을 스프레드한 뒤 파서 옵션 오버라이드와 ignores 항목 두 가지를 추가하여 최종 배열을 구성한다. ESLint는 이 배열을 순서대로 병합하여 `a2ui_explorer` 패키지 내 TypeScript/TSX 파일을 검사한다.
