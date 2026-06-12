# renderers/react/tsup.config.ts

## 개요

`@a2ui/react` 패키지의 번들 빌드를 구성하는 tsup 설정 파일이다. 두 개의 독립된 빌드 구성을 배열로 내보내며, 첫 번째는 컴포넌트 코드 + TypeScript 타입 선언(`dts`)을 포함하는 메인 빌드이고, 두 번째는 스타일 진입점만을 위한 별도 빌드로 타입 선언을 생략한다. `symlink` 해석 문제를 피하기 위해 스타일 빌드는 `dts: false`로 설정된다.

## 의존성

### 외부 패키지
- `tsup` — `defineConfig`

### 저장소 내부 모듈
없음 (빌드 설정 파일).

## Exports

- `default`: `defineConfig`로 생성된 설정 배열 (tsup 규약에 따른 default export)

## 상세 명세

### 첫 번째 빌드 구성 (메인 코드 + DTS)

| 항목 | 값 |
|------|-----|
| `entry` | `{ index: 'src/index.ts', 'v0_8/index': 'src/v0_8/index.ts', 'v0_9/index': 'src/v0_9/index.ts' }` |
| `format` | `['esm', 'cjs']` |
| `dts` | `true` — 타입 선언 파일 생성 |
| `splitting` | `true` — 코드 스플리팅 활성화 |
| `sourcemap` | `true` |
| `clean` | `true` — 빌드 전 출력 디렉토리 정리 |
| `treeshake` | `true` |
| `external` | `['react', 'react-dom', 'markdown-it']` — 번들에서 제외 |
| `esbuildOptions` | `options.jsx = 'automatic'` — React 17+ JSX 자동 변환 활성화 |

세 개의 진입점을 정의한다: 루트 `index`, `v0_8/index`, `v0_9/index`. 각각 ESM과 CJS 형식으로 출력된다. `react`, `react-dom`, `markdown-it`은 피어 의존성으로 취급하여 번들에 포함하지 않는다.

### 두 번째 빌드 구성 (스타일 진입점)

| 항목 | 값 |
|------|-----|
| `entry` | `{ 'styles/index': 'src/styles/index.ts', 'v0_8/styles/index': 'src/v0_8/styles/index.ts' }` |
| `format` | `['esm', 'cjs']` |
| `dts` | `false` — 타입 선언 생성 안 함 (symlink 해석 문제 방지) |
| `splitting` | `false` |
| `sourcemap` | `true` |
| `clean` | `false` — 첫 번째 빌드 결과물 덮어쓰지 않음 |
| `treeshake` | `true` |
| `external` | `['@a2ui/lit']` — Lit 스타일 패키지는 외부로 취급 |

스타일 진입점은 두 개다: `styles/index`(전역)와 `v0_8/styles/index`(버전별). `clean: false`로 설정하여 첫 번째 빌드가 생성한 출력물을 유지한다.

## 동작 흐름

tsup은 이 파일을 읽어 두 번의 번들링을 순차 실행한다. 첫 번째 빌드가 `dist/` 디렉토리를 정리(`clean: true`)하고 메인 코드를 생성한 뒤, 두 번째 빌드가 기존 결과물을 보존하면서(`clean: false`) 스타일 파일만 추가로 빌드한다.
