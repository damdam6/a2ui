# renderers/web_core/src/v0_8/events/index.ts

## 개요

`events` 디렉토리의 진입점(barrel) 파일로, 하위 모듈들의 모든 export를 재수출(re-export)해 외부에 단일 진입점을 제공한다.

## 의존성

### 외부 패키지
- (없음)

### 저장소 내부 모듈
- [`./base.js`](./base.ts.md) — `BaseEventDetail`
- [`./validation-event.js`](./validation-event.ts.md) — `ValidationEventDetail`, `A2UIValidationEvent`

## Exports

이 파일은 직접 정의하는 항목 없이 하위 모듈의 모든 공개 항목을 그대로 재수출한다:

- `export * from './base.js'` → `BaseEventDetail`
- `export * from './validation-event.js'` → `ValidationEventDetail`, `A2UIValidationEvent`

## 동작 흐름

런타임 코드가 없는 순수 re-export 파일이다. 상위 모듈에서 `import { A2UIValidationEvent } from '.../events/index.js'`와 같이 단일 경로로 모든 이벤트 관련 항목을 임포트할 수 있다.
