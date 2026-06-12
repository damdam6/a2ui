# renderers/react/visual-parity/playwright.config.ts

## 개요

React와 Lit 렌더러 간의 시각적 동등성(visual parity)을 검증하는 Playwright 테스트 설정 파일이다. 두 렌더러의 개발 서버를 동시에 기동하고, 동일한 컴포넌트를 각 렌더러로 렌더링한 스크린샷을 비교하는 테스트 환경을 구성한다. CI 환경과 로컬 환경에서 서로 다른 동작(재시도 횟수, 워커 수, 서버 재사용 여부)을 적용한다.

## 의존성

### 외부 패키지

- `@playwright/test` — `defineConfig`, `devices`

### 저장소 내부 모듈

없음.

## Exports

| 이름 | 종류 | 설명 |
|---|---|---|
| `default` | Playwright 설정 객체 | `defineConfig()`로 감싼 전체 테스트 설정 |

## 상세 명세

### 기본 내보내기 (Playwright 설정 객체)

`defineConfig(config)`를 통해 작성된 Playwright 전역 설정이다.

#### 기본 테스트 설정

| 옵션 | 값 | 설명 |
|---|---|---|
| `testDir` | `'./tests'` | 테스트 파일 탐색 루트 디렉토리 |
| `fullyParallel` | `true` | 테스트 파일 내 테스트도 병렬 실행 |
| `forbidOnly` | `!!process.env.CI` | CI 환경에서 `.only` 사용 시 오류 처리 |
| `retries` | CI: `2`, 로컬: `0` | 실패 시 재시도 횟수 |
| `workers` | CI: `1`, 로컬: `undefined`(기본값) | 병렬 워커 수 |

#### `reporter`

두 개의 리포터를 동시에 사용한다.
- `['html', {open: 'never'}]` — HTML 리포트 생성, 자동 열기 비활성화
- `['list']` — 터미널 목록형 출력

#### `use` (전역 테스트 옵션)

- `trace: 'on-first-retry'` — 첫 번째 재시도 시에만 트레이스 기록
- `screenshot: 'only-on-failure'` — 실패 시에만 스크린샷 저장

#### `projects` (브라우저 프로젝트)

단일 프로젝트만 정의한다.
- `name: 'chromium'`
- `use`: `devices['Desktop Chrome']` 설정 확산(spread) + 뷰포트 `{width: 1280, height: 720}` 오버라이드

#### `webServer` (개발 서버 기동)

두 서버를 배열로 정의하여 테스트 전에 모두 기동한다.

| 항목 | React 서버 | Lit 서버 |
|---|---|---|
| `command` | `yarn dev:react` | `yarn dev:lit` |
| `url` | `http://localhost:5001` | `http://localhost:5002` |
| `reuseExistingServer` | `!process.env.CI` | `!process.env.CI` |
| `timeout` | `120000` ms | `120000` ms |

로컬 환경에서는 이미 실행 중인 서버를 재사용(`reuseExistingServer: true`)하고, CI에서는 항상 새로 기동한다.

#### `expect.toHaveScreenshot`

시각적 비교의 기본 임계값을 설정한다.
- `maxDiffPixels: 0` — 기본적으로 픽셀 차이를 허용하지 않음(개별 테스트에서 오버라이드 가능)
- `threshold: 0.1` — 픽셀당 색상 차이 허용 비율 10%

## 동작 흐름

1. Playwright CLI가 이 파일을 로드하여 설정을 파싱한다.
2. `webServer` 설정에 따라 `yarn dev:react`(포트 5001)와 `yarn dev:lit`(포트 5002)를 각각 기동한다. 서버 준비 확인은 각 `url`에 대한 HTTP 응답으로 판단한다.
3. `./tests` 디렉토리의 테스트 파일을 Chromium 프로젝트 설정으로 실행한다.
4. 테스트는 두 서버에 각각 접근하여 동일 픽스처를 렌더링한 뒤 `toHaveScreenshot()`으로 스크린샷을 비교한다.
5. 결과는 HTML 리포트와 터미널 목록으로 출력된다.
