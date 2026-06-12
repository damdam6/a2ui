# specification/v0_9/eval/src/logger.ts

## 개요

Winston 기반의 로거 싱글턴을 생성하고 내보내는 파일이다. 초기에는 콘솔 전용으로 동작하며, `setupLogger` 함수를 호출하면 콘솔 로그 레벨을 사용자 지정 값으로 변경하고 선택적으로 파일 트랜스포트를 추가할 수 있다. 파일 트랜스포트는 항상 `debug` 레벨 이상의 모든 로그를 기록하는 반면, 콘솔 트랜스포트는 호출 시 전달된 `logLevel` 인자를 따른다.

## 의존성

### 외부 패키지
- `winston` — 로거 생성 및 트랜스포트 관리
- `path` (Node.js 내장) — 파일 경로 조합

### 저장소 내부 모듈
없음

## Exports

| 이름 | 종류 | 설명 |
|------|------|------|
| `logger` | 상수 (winston.Logger 인스턴스) | 전역 로거 싱글턴 |
| `setupLogger` | 함수 | 로거의 레벨과 파일 트랜스포트를 초기화 |

## 상세 명세

### 모듈 수준 변수

#### `fileTransport: winston.transport | null`
파일 트랜스포트 참조를 보관하는 모듈 수준 변수. 초기값은 `null`. `setupLogger` 호출 시 기존 트랜스포트를 제거하고 새로 교체하기 위해 사용한다.

#### `consoleTransport`
`winston.transports.Console` 인스턴스. 설정은 다음과 같다.
- `level`: 초기값 `'info'` (이후 `setupLogger`에서 덮어씀)
- `format`: `winston.format.combine`으로 두 포매터를 적용한다.
  1. `winston.format.colorize()` — 레벨 텍스트에 색상 적용
  2. `winston.format.printf` — 출력 문자열을 `\r\x1b[K${timestamp} [${level}]: ${message}` 형태로 구성한다. `\r`로 커서를 줄 시작으로 이동하고 `\x1b[K`로 커서 이후 내용을 지워 진행 표시줄을 덮어쓰지 않도록 한다.

### `logger` (export)

`winston.createLogger`로 생성한 싱글턴. 전역 레벨은 `'debug'`로 설정되어 모든 레벨의 로그가 트랜스포트까지 전달된다. 공유 포맷은 `winston.format.timestamp()`와 `winston.format.printf`의 조합으로 `${timestamp} [${level}]: ${message}` 형태로 출력한다. 초기 트랜스포트 배열에는 `consoleTransport`만 포함된다.

### `setupLogger(outputDir: string | undefined, logLevel: string): void` (export)

로거의 런타임 설정을 변경하는 함수.

1. `logger.level`을 `'debug'`로 강제 설정하여 파일 트랜스포트가 상세 로그를 놓치지 않도록 한다.
2. `consoleTransport.level`을 인자로 받은 `logLevel`로 업데이트한다.
3. `fileTransport`가 이미 존재하면 `logger.remove(fileTransport)`로 제거하고 `fileTransport`를 `null`로 초기화한다.
4. `outputDir`가 falsy가 아닌 경우, `path.join(outputDir, 'output.log')` 경로에 새 `winston.transports.File`을 생성한다. 이 파일 트랜스포트는 레벨 `'debug'`, 포맷은 `winston.format.combine(winston.format.timestamp(), winston.format.json())`을 사용한다. 생성된 트랜스포트를 `fileTransport`에 저장하고 `logger.add`로 등록한다.

## 동작 흐름

모듈 로드 시 `consoleTransport`와 `logger`가 즉시 생성되어 전역 싱글턴으로 유지된다. 애플리케이션 초기화 단계에서 `setupLogger`를 한 번 호출하면 콘솔 레벨이 조정되고 필요에 따라 파일 출력이 추가된다. 이후 어느 모듈에서든 `logger`를 import하여 동일한 인스턴스를 공유한다.
