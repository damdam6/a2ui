# renderers/react/vitest.config.ts

## 개요

`renderers/react` 패키지의 Vitest 단위 테스트 설정 파일이다. jsdom 환경, 전역 API 활성화, 커버리지 설정, 그리고 `@a2ui/react` 관련 경로 별칭(alias)을 정의한다.

## 의존성

### 외부 패키지
- `vitest/config` — `defineConfig`
- `path` — Node.js 표준 모듈 (`path.resolve`)

### 저장소 내부 모듈
없음

## Exports

- `default` — `defineConfig(...)` 반환값 (Vitest 설정 객체)

## 상세 명세

### `export default defineConfig({...})`

#### `test.globals`
값: `true`. `describe`, `it`, `expect` 등을 import 없이 전역에서 사용할 수 있게 한다.

#### `test.environment`
값: `'jsdom'`. 브라우저 환경을 시뮬레이션하는 jsdom을 테스트 환경으로 사용한다. React 컴포넌트 렌더링 테스트에 필요하다.

#### `test.setupFiles`
값: `['./tests/setup.ts']`. 각 테스트 파일 실행 전에 `tests/setup.ts`를 먼저 실행한다.

#### `test.include`
값: `['tests/**/*.test.{ts,tsx}']`. `tests/` 디렉터리 아래의 `.test.ts` 또는 `.test.tsx` 파일을 테스트 대상으로 인식한다.

#### `test.coverage`
- `provider: 'v8'` — V8 내장 커버리지 엔진 사용
- `reporter: ['text', 'json', 'html']` — 터미널 텍스트, JSON, HTML 형식으로 커버리지 리포트 생성
- `include: ['src/**/*.{ts,tsx}']` — `src/` 아래의 `.ts`, `.tsx` 파일만 커버리지 측정 대상
- `exclude: ['src/**/*.d.ts', 'src/index.ts']` — 타입 선언 파일과 barrel 파일(`src/index.ts`)은 제외

#### `resolve.alias`
프로세스 현재 작업 디렉터리(`process.cwd()`)를 기준으로 소스 파일 경로를 패키지 임포트 경로로 매핑한다. 이를 통해 테스트 코드가 빌드 없이 소스를 직접 참조한다.

| alias 키 | 해석 경로 |
|---|---|
| `@` | `src/v0_8` |
| `@a2ui/react/v0_9` | `src/v0_9/index.ts` |
| `@a2ui/react/v0_8` | `src/v0_8/index.ts` |
| `@a2ui/react/styles` | `src/styles/index.ts` |
| `@a2ui/react` | `src/index.ts` |

## 동작 흐름

Vitest가 이 파일을 읽어 테스트 환경을 구성한다. `tests/setup.ts`를 먼저 실행한 뒤 `tests/**/*.test.{ts,tsx}` 파일을 jsdom 환경에서 실행한다. alias 덕분에 `@a2ui/react` 패키지를 import하면 빌드 결과물이 아닌 `src/index.ts` 소스를 직접 참조한다.
