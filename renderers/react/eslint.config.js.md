# renderers/react/eslint.config.js

## 개요

`renderers/react` 패키지 전체에 적용되는 ESLint flat config 파일이다. Google TypeScript 스타일 가이드(gts), React 및 React Hooks 플러그인, Prettier 포매팅 설정을 통합한다. 하위 패키지(`a2ui_explorer`)가 이 파일을 부모 설정으로 상속한다.

## 의존성

### 외부 패키지
- `@eslint/js` — 기본 JavaScript 규칙 (`js`)
- `typescript-eslint` — TypeScript ESLint 통합 (`tseslint`)
- `eslint-plugin-react` — React 린트 규칙
- `eslint-plugin-react-hooks` — React Hooks 린트 규칙
- `eslint-config-prettier` — Prettier와 충돌하는 규칙 비활성화
- `gts` — Google TypeScript 스타일 가이드 preset

### 저장소 내부 모듈
없음

## Exports

기본(default) export: `tseslint.config(...)` 반환값 (ESLint flat config 배열)

## 상세 명세

### default export (배열)

`tseslint.config()` 헬퍼로 구성된 설정 배열이다. 구성 순서는 다음과 같다.

1. **`...gts`**: Google TypeScript 스타일 가이드 전체 규칙을 스프레드로 포함한다.

2. **React/TypeScript 설정 블록** (`files: ['**/*.{ts,tsx}']`):
   - `plugins`: `react` (reactPlugin), `react-hooks` (reactHooksPlugin)
   - `languageOptions.parserOptions.ecmaFeatures.jsx`: `true` — JSX 파싱 활성화
   - `settings.react.version`: `'detect'` — React 버전 자동 감지
   - **규칙**:
     - `react-hooks/rules-of-hooks`: `'error'`
     - `react-hooks/exhaustive-deps`: `'warn'`
     - `react/jsx-uses-react`: `'off'` (새 JSX 변환 사용으로 불필요)
     - `react/react-in-jsx-scope`: `'off'` (새 JSX 변환 사용으로 불필요)
     - `react/prop-types`: `'off'` (TypeScript로 대체)
     - `react/display-name`: `'warn'`
     - `@typescript-eslint/no-unused-vars`: `'error'` (단, `^_` 패턴의 인수/변수는 무시)
     - `@typescript-eslint/no-explicit-any`: `'warn'`
     - `@typescript-eslint/consistent-type-imports`: `'warn'` (inline type-imports 방식 선호)
     - `no-console`: `['warn', {allow: ['warn', 'error']}]`
     - `prefer-arrow-callback`: `'off'`

3. **테스트 파일 완화 규칙** (`files: ['tests/**/*.{ts,tsx}', '**/*.test.{ts,tsx}']`):
   - `@typescript-eslint/no-explicit-any`: `'off'`
   - `no-console`: `'off'`

4. **`prettierConfig`**: Prettier와 충돌하는 모든 규칙을 비활성화한다. 반드시 마지막에 위치해야 한다.

5. **무시 경로**: `ignores: ['dist/**', 'node_modules/**', 'visual-parity/**', '**/*.d.ts']`

## 동작 흐름

ESLint가 `renderers/react` 디렉터리에서 실행되면 이 파일을 기본 설정으로 로드한다. `a2ui_explorer/eslint.config.js`는 이 파일을 `parentConfig`로 import하여 TypeScript 파서 옵션만 추가로 확장한다.
