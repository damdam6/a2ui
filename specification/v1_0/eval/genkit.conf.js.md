# specification/v1_0/eval/genkit.conf.js

## 개요

Genkit 프레임워크의 전역 설정을 구성하는 파일이다. `@genkit-ai/google-genai` 플러그인을 등록하고, 디버그 로그 레벨과 트레이싱/메트릭 기능을 활성화한 Genkit 인스턴스를 기본 내보내기로 제공한다. 이 파일은 Genkit CLI(`genkit start` 등)가 실행될 때 자동으로 읽히는 설정 진입점이다.

## 의존성

### 외부 패키지
- `@genkit-ai/google-genai` — Google AI(Gemini) 연동 Genkit 플러그인
- `genkit` — Genkit 코어, `configure` 함수 제공

### 저장소 내부 모듈
없음

## Exports

| 이름 | 종류 | 설명 |
|------|------|------|
| `default` | Genkit 설정 인스턴스 | `configure(...)` 호출 결과를 default export로 내보냄 |

## 상세 명세

### `configure(options)` 호출 (default export)

`genkit` 패키지의 `configure` 함수를 호출하여 Genkit 런타임 설정 객체를 생성하고 default export로 내보낸다.

**옵션 값:**
- `plugins`: `[googleAI()]` — Google AI 플러그인을 인수 없이 초기화. API 키는 환경 변수(`GEMINI_API_KEY` 등)에서 자동 탐색.
- `logLevel`: `'debug'` — 가장 상세한 로그 수준을 활성화.
- `enableTracingAndMetrics`: `true` — OpenTelemetry 기반 트레이싱과 메트릭 수집을 활성화.

## 동작 흐름

모듈 로드 시 `configure`가 즉시 실행되어 Genkit 전역 설정이 등록된다. 이 설정은 Genkit 개발 UI(`genkit start`) 또는 플로우 러너가 참조하는 기본 런타임 설정으로 사용된다. 실제 애플리케이션 코드(`ai.ts`)는 이 파일과 별도로 `genkit()` 인스턴스를 직접 생성하므로 런타임 실행 경로에서 이 파일을 직접 import하지는 않는다.
