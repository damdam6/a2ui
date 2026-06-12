# specification/v0_9/eval/genkit.conf.js

## 개요

Genkit 개발 서버 및 도구(예: `genkit start`)를 실행할 때 사용하는 Genkit 프레임워크의 전역 설정 파일이다. `@genkit-ai/google-genai` 플러그인을 등록하고, 로그 레벨을 `debug`로, 트레이싱 및 메트릭 수집을 활성화한 상태로 구성한다. 이 파일은 소스 코드에서 직접 `import`되지 않고, Genkit CLI가 자동으로 탐색하여 로드하는 설정 진입점이다.

## 의존성

### 외부 패키지

- `@genkit-ai/google-genai` — `googleAI` 플러그인 팩토리 함수
- `genkit` — `configure` 함수

### 저장소 내부 모듈

없음

## Exports

- `default` — `configure(...)` 호출의 반환값인 Genkit 설정 인스턴스

## 상세 명세

### 기본 내보내기 (default export)

`configure` 함수를 다음 옵션으로 호출한 결과를 default export로 내보낸다.

- `plugins`: `[googleAI()]` — Google AI(Gemini) 플러그인을 기본 옵션으로 초기화
- `logLevel`: `'debug'` — 모든 디버그 로그 출력 활성화
- `enableTracingAndMetrics`: `true` — OpenTelemetry 기반 분산 트레이싱 및 메트릭 수집 활성화

## 동작 흐름

파일이 평가될 때 `configure(...)` 가 즉시 호출되어 Genkit 전역 설정 객체를 생성하고, 이를 default export로 제공한다. Genkit CLI(`genkit start`, `genkit flow:run` 등)는 프로젝트 루트에서 이 파일을 자동으로 찾아 Genkit 런타임 환경을 초기화한다.
