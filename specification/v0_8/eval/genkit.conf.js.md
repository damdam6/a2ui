# specification/v0_8/eval/genkit.conf.js

## 개요

이 파일은 Genkit 프레임워크의 기본 설정을 담당하는 JavaScript 설정 모듈이다. Google AI 플러그인을 등록하고, 디버그 로그 수준과 트레이싱/메트릭 활성화 옵션을 포함한 Genkit 인스턴스를 구성하여 기본 export로 내보낸다. 개발 환경에서 Genkit 개발 서버(`genkit start`)가 이 파일을 자동으로 로드하여 플러그인 및 플로우 설정에 활용한다.

## 의존성

### 외부 패키지
- `@genkit-ai/google-genai` — `googleAI` 플러그인 팩토리 함수
- `genkit` — `configure` 함수 (Genkit 인스턴스 구성)

### 저장소 내부 모듈
없음

## Exports

| 이름 | 종류 | 설명 |
|------|------|------|
| `default` | 상수 (Genkit 인스턴스) | `configure()`가 반환하는 구성된 Genkit 인스턴스 |

## 상세 명세

### default export (Genkit 구성 인스턴스)

`configure()` 함수를 호출하여 반환된 Genkit 인스턴스를 기본 export로 내보낸다. 호출 시 전달하는 설정 객체는 다음 세 필드를 포함한다.

- `plugins`: `[googleAI()]` — 인수 없이 호출한 `googleAI()` 팩토리를 배열로 전달한다. 이 플러그인은 환경 변수 `GEMINI_API_KEY`를 자동으로 읽어 Google Generative AI API에 접근한다.
- `logLevel`: `'debug'` — Genkit 내부 로그를 debug 수준으로 출력한다.
- `enableTracingAndMetrics`: `true` — OpenTelemetry 기반 트레이싱과 메트릭 수집을 활성화한다.

## 동작 흐름

모듈이 로드되는 시점에 `configure()`가 즉시 실행되어 Genkit 인스턴스를 생성하고 기본 export로 노출한다. 별도의 함수 호출 없이 import 만으로 설정이 완료된다. 이 파일은 `genkit start` 명령이 사용하는 진입점으로, `flows.ts` 등 다른 파일은 별도의 `genkit()` 인스턴스를 자체적으로 생성하므로 이 설정 파일은 개발 서버 전용으로 분리되어 있다.
