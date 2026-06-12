# renderers/angular/a2ui_explorer/src/app/tests/utils/index.ts

## 개요

테스트 유틸리티 모듈의 배럴(barrel) 파일이다. 같은 디렉토리의 `test_utils.ts`에서 내보내는 모든 심볼을 그대로 재내보내어, 테스트 파일이 `'../utils'` 단일 경로로 모든 헬퍼에 접근할 수 있도록 한다. 자체 로직 없이 `export *` 하나로만 구성된다.

## 의존성

### 저장소 내부 모듈
- [`./test_utils`](./test_utils.ts.md) — 전체 재내보내기

## Exports

`test_utils.ts`의 모든 공개 심볼을 그대로 재내보낸다:
- `Version` (열거형 재내보내기)
- `loadExample` (비동기 함수)
- `wait` (함수)
- `getCanvas` (함수)
- `waitForCondition` (비동기 함수)

## 동작 흐름

`export * from './test_utils'` 단일 구문으로 구성된다. 런타임 실행 로직 없음.
