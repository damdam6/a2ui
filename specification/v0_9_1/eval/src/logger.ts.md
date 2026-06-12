# specification/v0_9_1/eval/src/logger.ts

## 개요

이 파일은 평가 도구 전반에서 사용하는 공유 `winston` 로거 인스턴스를 초기화하고 내보낸다. 초기에는 콘솔 전송만 활성화되어 있으며, `setupLogger`를 호출하면 콘솔 로그 레벨을 조정하고 선택적으로 파일 전송을 추가할 수 있다. 진행 표시줄이 표시되는 환경에서 콘솔 출력이 줄바꿈 없이 덮어쓰여지도록 ANSI 이스케이프 시퀀스로 현재 줄을 지우는 포맷터를 사용한다.

## 의존성

### 외부 패키지
- `winston` — 로깅 프레임워크
- `path` — Node.js 내장 경로 모듈

### 저장소 내부 모듈
없음

## Exports

| 이름 | 종류 |
|---|---|
| `logger` | `winston.Logger` 상수 (싱글턴 인스턴스) |
| `setupLogger` | 함수 |

## 상세 명세

### 모듈 레벨 상태

**`fileTransport: winston.transport | null`** (모듈 private)
- 현재 활성화된 파일 전송 인스턴스를 추적하는 변수. 초기값 `null`.
- `setupLogger`가 호출될 때마다 이전 파일 전송을 제거하고 교체하기 위해 사용한다.

**`consoleTransport`** (모듈 private)
- `winston.transports.Console` 인스턴스.
- 초기 레벨: `'info'`.
- 포맷: `colorize()` + 커스텀 `printf`. `printf` 포맷터는 `\r\x1b[K` (현재 줄 첫 칸으로 이동 후 줄 끝까지 지우기)를 메시지 앞에 붙여 진행 표시줄을 덮어쓰지 않도록 한다. 출력 형식: `` `\r\x1b[K${timestamp} [${level}]: ${message}` ``.

### `logger` (export)

`winston.createLogger`로 생성된 싱글턴 인스턴스.

- 전역 레벨: `'debug'` (모든 레벨이 전송 레이어까지 흘러내려가도록 설정, 실제 필터링은 각 전송이 담당).
- 포맷: `timestamp()` + `printf`. 출력 형식: `` `${timestamp} [${level}]: ${message}` `` (ANSI 제거, 파일 저장에 적합한 일반 텍스트).
- 전송: 초기에는 `consoleTransport` 하나만 포함.

### `setupLogger(outputDir: string | undefined, logLevel: string): void` (export)

호출 시 다음 단계를 순서대로 수행한다.

1. `logger.level`을 `'debug'`로 강제 설정하여 모든 로그가 전송 레이어에 도달하도록 한다.
2. `consoleTransport.level`을 `logLevel` 인자로 업데이트한다 (사용자가 지정한 로그 수준으로 콘솔 출력을 제한).
3. `fileTransport`가 이미 존재하면 `logger.remove(fileTransport)`로 제거하고 `null`로 초기화한다.
4. `outputDir`가 truthy이면 `path.join(outputDir, 'output.log')` 경로에 새 `winston.transports.File` 인스턴스를 생성한다.
   - 파일 전송 레벨: `'debug'` (항상 모든 로그를 파일에 기록).
   - 포맷: `timestamp()` + `json()` (구조화된 JSON 형식으로 저장).
   - 생성된 인스턴스를 `fileTransport`에 저장하고 `logger.add`로 등록한다.

## 동작 흐름

모듈 로드 시 콘솔 전송만 갖춘 `logger`가 즉시 생성되어 사용 가능 상태가 된다. 애플리케이션 시작 시 CLI 인자에서 출력 디렉토리와 로그 레벨을 파싱한 뒤 `setupLogger`를 한 번 호출하면, 콘솔 레벨을 조정하고 파일 로깅을 활성화한다. 이후 모든 모듈은 `import {logger}`로 동일한 싱글턴을 공유하므로, `setupLogger` 호출 이후의 설정 변경이 전체에 즉시 반영된다.
