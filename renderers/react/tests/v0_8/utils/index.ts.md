# renderers/react/tests/v0_8/utils/index.ts

## 개요

v0.8 테스트 유틸리티의 배럴(barrel) 파일이다. 동일 디렉토리 내 세 모듈(`render`, `messages`, `assertions`)을 `export *` 형태로 재내보내어, 테스트 파일이 단일 경로(`../../utils`)로 모든 헬퍼를 임포트할 수 있게 한다.

## 의존성

### 외부 패키지
없음.

### 저장소 내부 모듈
- [`./render`](./render.tsx.md) — `TestRenderer`, `TestWrapper`
- [`./messages`](./messages.ts.md) — `createSimpleMessages`, `createSurfaceUpdate`, `createBeginRendering`, `createDataModelUpdate`, `createDeleteSurface`, `createDataModelUpdateSpec`
- [`./assertions`](./assertions.ts.md) — `getMockCallArg`, `getElement`

## Exports

위 세 모듈의 모든 공개 심볼을 통과(re-export)한다.

## 동작 흐름

임포트 진입점 역할만 하며 자체 로직은 없다. 세 `export *` 구문을 선언 순서대로 실행하고, 각 모듈의 named export를 모두 노출한다.
