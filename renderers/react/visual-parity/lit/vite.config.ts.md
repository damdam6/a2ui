# renderers/react/visual-parity/lit/vite.config.ts

## 개요

Lit 렌더러 시각적 동등성 테스트 서버(`dev:lit`)의 Vite 빌드 설정 파일이다. 개발 서버 포트, 의존성 최적화 전략, esbuild 트랜스파일 타깃을 구성한다. 특히 `@a2ui/lit`와 관련 패키지들을 사전 번들링에서 제외하여 중복 모듈 인스턴스 문제를 방지한다.

## 의존성

### 외부 패키지

- `vite` — `defineConfig`

### 저장소 내부 모듈

없음.

## Exports

| 이름 | 종류 | 설명 |
|---|---|---|
| `default` | Vite 설정 객체 | `defineConfig()`로 감싼 설정 |

## 상세 명세

### 기본 내보내기 (Vite 설정 객체)

`defineConfig(config)`를 통해 타입 안전성을 확보하며 다음 옵션을 설정한다.

#### `root`

`'lit'` — Vite가 `index.html`을 찾는 루트 디렉토리를 `lit/` 서브디렉토리로 지정한다.

#### `server`

- `port`: `5002` — 개발 서버 포트 번호. React 렌더러 서버(`5001`)와 구별된다.
- `strictPort`: `true` — 지정 포트가 이미 사용 중이면 다른 포트로 대체하지 않고 오류로 처리한다.

#### `optimizeDeps`

- `force: true` — 시작 시 항상 재최적화를 수행한다. `file:` 의존성 재빌드 후 504 오류를 방지하기 위한 설정이다.
- `exclude: ['@a2ui/lit', 'markdown-it', 'clsx', 'signal-utils/array', 'signal-utils/map', 'signal-utils/object', 'signal-utils/set']` — 해당 패키지들을 사전 번들링(pre-bundle) 대상에서 제외한다. `@a2ui/lit`와 그 의존 패키지들을 제외하는 이유는 중복 모듈 인스턴스가 생성될 경우 Lit 커스텀 엘리먼트 등록 충돌이 발생하기 때문이다.

#### `esbuild`

- `target: 'es2022'` — TC39 Stage 3 데코레이터(Lit `@customElement`, `@property`, `@provide` 등)를 올바르게 처리하기 위해 ES2022 이상을 타깃으로 설정한다.

## 동작 흐름

`vite` CLI가 이 파일을 로드하면 `defineConfig`가 설정 객체를 반환하고, Vite는 해당 설정에 따라 `lit/` 디렉토리를 루트로 삼아 포트 `5002`에서 개발 서버를 시작한다. Playwright `playwright.config.ts`의 `webServer` 설정이 `yarn dev:lit` 명령으로 이 서버를 기동한다.
