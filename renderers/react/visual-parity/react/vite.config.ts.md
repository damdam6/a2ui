# renderers/react/visual-parity/react/vite.config.ts

## 개요

비주얼 패리티 React 앱(포트 5001)을 위한 Vite 빌드 설정 파일이다. React 플러그인 등록, 개발 서버 포트 고정, 의존성 최적화 제어, React 중복 인스턴스 방지를 위한 alias 설정을 담당한다.

## 의존성

### 외부 패키지
- `vite` — `defineConfig`
- `@vitejs/plugin-react` — `react` (Vite React 플러그인)
- `path` — Node.js 표준 모듈 (`path.dirname`)
- `module` — Node.js 표준 모듈 (`createRequire`)

### 저장소 내부 모듈
없음

## Exports

- `default` — `defineConfig(...)` 반환값 (Vite 설정 객체)

## 상세 명세

### 모듈 최상위 코드

`createRequire(import.meta.url)`로 ESM 환경에서 CommonJS `require` 함수를 생성한다. 이는 패키지의 실제 설치 경로를 얻기 위해 `require.resolve`를 사용할 때 필요하다.

### `export default defineConfig({...})`

#### `plugins`
`react()` 플러그인 하나만 포함한다. JSX 변환 및 Fast Refresh를 제공한다.

#### `root`
값: `'react'`. Vite가 `index.html`을 찾을 루트 디렉터리를 `react/` 하위로 지정한다. 이 파일 자체는 `visual-parity/` 아래에 있으므로 실제 경로는 `visual-parity/react/`가 된다.

#### `server`
- `port: 5001` — 개발 서버 포트 번호
- `strictPort: true` — 5001번 포트가 이미 사용 중이면 다른 포트로 폴백하지 않고 오류를 낸다

#### `optimizeDeps`
- `force: true` — 서버 시작 시 항상 재최적화. `file:` 의존성 재빌드 후 504 에러를 방지한다.
- `include: ['@a2ui/web_core/styles/index', '@a2ui/web_core/data/model-processor']` — `web_core`의 서브패스 import를 사전 번들링한다. 지연 발견 시 발생하는 "optimized dependencies changed. reloading" 현상을 막는다.
- `exclude: ['@a2ui/react', '@a2ui/lit', 'markdown-it', 'clsx', 'signal-utils/array', 'signal-utils/map', 'signal-utils/object', 'signal-utils/set']` — 해당 패키지들은 사전 번들링 대상에서 제외한다.

#### `resolve.alias`
- `react` → `path.dirname(require.resolve('react/package.json'))` — 로컬에 설치된 React 패키지 디렉터리로 고정
- `react-dom` → `path.dirname(require.resolve('react-dom/package.json'))` — 로컬에 설치된 react-dom 디렉터리로 고정

이 두 alias는 링크된 패키지(예: `@a2ui/react`)가 자신의 `node_modules`에서 별도의 React 인스턴스를 참조할 때 발생하는 "Invalid hook call" 오류를 방지한다.

## 동작 흐름

Vite가 이 파일을 읽어 설정 객체를 구성한다. `root: 'react'` 설정으로 인해 `visual-parity/react/`의 `index.html`이 앱 진입점이 된다. 개발 서버는 항상 5001번 포트에서만 실행되며, A2UI 관련 의존성 일부는 사전 번들링에서 제외되어 Vite가 직접 처리한다.
