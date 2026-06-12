# specification/v0_9_1/eval/genkit.conf.js

## 개요

Genkit 프레임워크의 전역 설정 파일이다. Google AI 플러그인을 등록하고, 로그 레벨을 `'debug'`로 설정하며, 트레이싱과 메트릭 수집을 활성화한 Genkit 인스턴스를 기본 내보내기로 제공한다. 이 파일은 Genkit 개발 서버(`genkit start`) 실행 시 자동으로 로드되는 설정 진입점 역할을 한다.

## 의존성

### 외부 패키지
- `@genkit-ai/google-genai` — `googleAI` 플러그인 팩토리
- `genkit` — `configure` 함수

### 저장소 내부 모듈
없음.

## Exports

- (기본 내보내기) `configure(...)` 호출의 반환값 — Genkit 설정 객체

## 상세 명세

### 기본 내보내기 (configure 호출)

`configure` 함수에 아래 옵션 객체를 전달한 결과를 `export default`로 내보낸다.

- `plugins`: `[googleAI()]` — 인자 없이 호출하여 Google AI 플러그인을 기본값으로 초기화
- `logLevel`: `'debug'` — 모든 디버그 메시지 출력
- `enableTracingAndMetrics`: `true` — 요청 트레이싱 및 메트릭 수집 활성화

## 동작 흐름

모듈 로드 시 `configure`가 즉시 실행되며, 그 결과 객체가 기본 내보내기가 된다. 런타임에서 이 파일을 import하거나 Genkit CLI가 자동 탐지하면 Google AI 플러그인이 등록된 전역 Genkit 인스턴스가 구성된다.
