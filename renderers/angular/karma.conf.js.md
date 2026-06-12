# renderers/angular/karma.conf.js

## 개요

로컬 개발 환경에서 사용하는 Karma 기본 설정 파일이다. Jasmine 테스트 프레임워크, Chrome 브라우저, HTML/커버리지 리포터를 포함한 전체 Karma 설정을 정의한다. 파일 변경 시 자동 재시작을 활성화하며, CI 안정성을 위한 타임아웃 설정도 포함한다.

## 의존성

### 외부 패키지
- `karma-jasmine` — Jasmine 프레임워크 플러그인
- `karma-chrome-launcher` — Chrome 브라우저 런처 플러그인
- `karma-jasmine-html-reporter` — Jasmine HTML 리포터 플러그인
- `karma-coverage` — 커버리지 리포터 플러그인
- `path` (Node.js 내장) — 커버리지 출력 경로 계산

### 저장소 내부 모듈
없음

## Exports

| 이름 | 종류 | 설명 |
|---|---|---|
| `module.exports` | 함수 `(config) => void` | Karma 설정 함수 |

## 상세 명세

### `module.exports = function(config)`

Karma가 호출하는 설정 함수다. `config.set()`으로 아래 전체 설정을 적용한다.

**프레임워크 및 플러그인:**
- `basePath`: `''` — 설정 파일 위치 기준 경로
- `frameworks`: `['jasmine']` — Jasmine 테스트 프레임워크 사용
- `plugins`: `karma-jasmine`, `karma-chrome-launcher`, `karma-jasmine-html-reporter`, `karma-coverage` 4개를 `require`로 나열

**Jasmine 클라이언트 설정:**
- `client.jasmine`: 빈 객체. `random: false`나 `seed` 등 추가 옵션을 여기에 설정할 수 있다는 주석이 있으나 현재 미설정.

**리포터 설정:**
- `jasmineHtmlReporter.suppressAll`: `true` — 중복 스택 트레이스 제거
- `coverageReporter`:
  - `dir`: `path.join(__dirname, './coverage/lib')` — 커버리지 출력 디렉토리
  - `subdir`: `'.'` — 서브디렉토리 없이 루트에 저장
  - `reporters`: `[{type: 'html'}, {type: 'text-summary'}]`
- `reporters`: `['progress', 'kjhtml']`

**브라우저 및 실행 설정:**
- `browsers`: `['Chrome']`
- `restartOnFileChange`: `true` — 파일 변경 시 자동 재시작

**네트워크 및 타임아웃 설정** (`karma-custom.conf.js`와 동일한 값):

| 설정 키 | 값 |
|---|---|
| `hostname` | `'127.0.0.1'` |
| `listenAddress` | `'127.0.0.1'` |
| `captureTimeout` | `210000` |
| `browserNoActivityTimeout` | `210000` |
| `browserDisconnectTimeout` | `10000` |
| `browserDisconnectTolerance` | `3` |

## 동작 흐름

Angular CLI(`ng test`)가 이 파일을 읽어 Karma 서버를 구성한다. 플러그인 등록 순서대로 Jasmine → Chrome → 리포터가 초기화된다. 테스트 실행 결과는 콘솔(`progress`)과 브라우저 HTML 페이지(`kjhtml`) 양쪽에 표시되며, 커버리지 데이터는 `./coverage/lib` 하위에 저장된다. CI 환경에서는 `karma-custom.conf.js`로 대체하거나 추가 설정을 병합하여 사용한다.
