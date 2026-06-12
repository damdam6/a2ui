# specification/v1_0/eval/src/dev.ts

## 개요

Genkit 개발 서버(`genkit start`) 실행 시 flow들을 등록하기 위한 진입점 파일이다. `generation_flow`와 `evaluation_flow` 모듈을 side-effect import하여 해당 flow들이 Genkit 런타임에 등록되도록 한다. 자체적으로 로직이나 export를 갖지 않는 진입점 역할만 수행한다.

## 의존성

### 저장소 내부 모듈
- [`./generation_flow`](./generation_flow.ts.md) — side-effect import (flow 등록 목적)
- [`./evaluation_flow`](./evaluation_flow.ts.md) — side-effect import (flow 등록 목적)

## Exports

없음

## 동작 흐름

이 파일이 Node.js에 의해 로드되면, import 문이 실행되면서 `generation_flow.ts`와 `evaluation_flow.ts` 각각의 최상위 코드(`ai.defineFlow(...)`)가 실행되어 해당 flow들이 Genkit 런타임 레지스트리에 등록된다. Genkit CLI는 이 파일을 개발 서버의 진입점으로 사용하여 등록된 flow 목록을 UI에 노출한다.
