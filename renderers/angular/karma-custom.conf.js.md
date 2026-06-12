# renderers/angular/karma-custom.conf.js

## 개요

CI/자동화 환경에서 사용되는 Karma 커스텀 설정 파일이다. 기본 `karma.conf.js`와 달리 네트워크 및 타임아웃 관련 설정만 재정의하며, 브라우저·프레임워크·플러그인 설정은 포함하지 않는다. 로컬 루프백 바인딩과 긴 타임아웃 값으로 원격 CI 환경의 불안정한 연결 문제를 방지한다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈
없음

## Exports

| 이름 | 종류 | 설명 |
|---|---|---|
| `module.exports` | 함수 `(config) => void` | Karma 설정 함수 |

## 상세 명세

### `module.exports = function(config)`

Karma 프레임워크가 호출하는 설정 함수다. `config.set()`에 다음 키-값만 설정한다.

| 설정 키 | 값 | 설명 |
|---|---|---|
| `hostname` | `'127.0.0.1'` | 서버 호스트명을 루프백으로 고정 |
| `listenAddress` | `'127.0.0.1'` | 리슨 주소를 루프백으로 고정 |
| `captureTimeout` | `210000` | 브라우저 캡처 타임아웃 3.5분(주석에 명시) |
| `browserNoActivityTimeout` | `210000` | 브라우저 무활동 타임아웃 3.5분 |
| `browserDisconnectTimeout` | `10000` | 브라우저 연결 끊김 감지 타임아웃 10초 |
| `browserDisconnectTolerance` | `3` | 연결 끊김 허용 횟수 |

## 동작 흐름

Karma 실행 시 이 설정 함수가 로드되면 `config.set`으로 네트워크·타임아웃 관련 설정만 덮어쓴다. 프레임워크·플러그인·리포터·브라우저 등의 설정은 기본 `karma.conf.js`에서 담당하며 이 파일은 이를 오버라이드하지 않는다.
