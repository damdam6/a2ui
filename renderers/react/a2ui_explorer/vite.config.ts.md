# renderers/react/a2ui_explorer/vite.config.ts

## 개요

`a2ui_explorer` 애플리케이션의 Vite 빌드 및 개발 서버 설정 파일이다. React 플러그인 적용, 개발 서버의 파일 서빙 루트 확장, 저장소 내부 패키지 경로 별칭(alias) 정의, Vitest 단위 테스트 환경 구성을 담당한다.

## 의존성

### 외부 패키지
- `vite` — `defineConfig`
- `@vitejs/plugin-react` — `react` 플러그인
- `path` (Node.js 내장) — `resolve`

### 저장소 내부 모듈
없음 (설정 파일)

## Exports

기본(default) export: Vite 설정 객체 (`defineConfig` 래핑)

## 상세 명세

### default export

`defineConfig({ plugins, server, resolve, test })` 형태의 설정 객체이다. `as any`로 캐스팅하여 TypeScript 엄격 검사를 우회한다.

**`plugins`**: `[react()]` — Babel 기반 React Fast Refresh 및 JSX 변환을 활성화한다.

**`server.fs.allow`**: `['../../../../']` — 개발 서버가 저장소 루트까지의 파일(특히 specification 디렉터리)을 서빙할 수 있도록 허용 범위를 확장한다.

**`resolve.alias`**: 저장소 내부 패키지를 소스 경로로 직접 매핑한다.

| 별칭 | 실제 경로 |
|---|---|
| `@a2ui/react/v0_9` | `../src/v0_9/index.ts` |
| `@a2ui/react/v0_8` | `../src/v0_8/index.ts` |
| `@a2ui/react/styles` | `../src/styles/index.ts` |
| `@a2ui/react` | `../src/index.ts` |
| `@a2ui/markdown-it` | `../../markdown/markdown-it/dist/src/markdown.js` |

모든 경로는 `resolve(__dirname, ...)` 형태로 절대 경로화된다.

**`test`**: Vitest 설정.
- `environment`: `'jsdom'` — 브라우저 DOM 환경 시뮬레이션
- `globals`: `true` — `describe`, `it`, `expect` 등 전역 API 사용
- `setupFiles`: `['./src/setupTests.ts']` — 각 테스트 파일 실행 전 jest-dom 매처 등록

## 동작 흐름

Vite CLI(`vite dev` 또는 `vite build`) 실행 시 이 파일을 로드하여 설정을 적용한다. 개발 서버는 React 플러그인으로 HMR을 지원하며, 패키지 별칭을 통해 `node_modules` 없이 모노레포 내부 패키지를 직접 참조한다. Vitest(`vitest run`) 실행 시에도 동일한 설정을 활용한다.
