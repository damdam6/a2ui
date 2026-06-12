# specification/v0_9/eval/src/dev.ts

## 개요

Genkit 개발 서버(`genkit start`) 실행 시 플로우들을 Genkit 런타임에 등록하기 위한 진입점 파일이다. 직접 실행 가능한 로직은 없으며, 사이드 이펙트로 플로우 모듈을 로드하는 역할만 한다.

## 의존성

### 외부 패키지

없음

### 저장소 내부 모듈

- [`./generation_flow`](./generation_flow.ts.md) — `componentGeneratorFlow` 등록 (사이드 이펙트)
- [`./evaluation_flow`](./evaluation_flow.ts.md) — `evaluationFlow` 등록 (사이드 이펙트)

## Exports

없음 (모든 import가 사이드 이펙트 전용)

## 동작 흐름

모듈이 로드되면 `generation_flow`와 `evaluation_flow`가 각각 import되어, 그 안의 `ai.defineFlow(...)` 호출이 실행된다. 이로써 두 플로우가 Genkit 런타임에 등록되어 `genkit start`로 실행된 개발 UI에서 탐색 및 테스트 가능한 상태가 된다.
