# specification/v0_8/eval/src/dev.ts

## 개요

이 파일은 Genkit 개발 서버를 위한 진입점 역할을 한다. `flows.ts`를 side-effect import하여 해당 모듈에 정의된 Genkit 플로우들이 등록되도록 한다. `genkit start --attach` 명령이 이 파일을 로드하면 `componentGeneratorFlow`를 포함한 모든 플로우가 개발 서버에 노출된다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈
- [`./flows`](./flows.ts.md) — side-effect import (플로우 등록 목적)

## Exports

없음

## 상세 명세

이 파일은 `import './flows';` 한 줄로 구성된다. 별도의 함수, 클래스, 상수를 정의하거나 export하지 않는다.

## 동작 흐름

모듈이 로드되는 즉시 `flows.ts`가 실행되고, 그 안에서 정의된 `ai` Genkit 인스턴스와 `componentGeneratorFlow`가 초기화된다. Genkit 개발 서버는 이 진입점을 통해 등록된 모든 플로우를 발견하고 개발 UI에서 직접 실행할 수 있게 된다.
