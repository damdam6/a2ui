# renderers/react/src/styles/index.ts

## 개요

패키지 루트에서 스타일을 임포트할 수 있는 진입점으로, `v0_8/styles/index`의 모든 스타일 심벌을 재수출한다. `postbuild.mjs`에 의해 `.d.ts`와 `.d.cts` 선언 파일이 `dist/styles/`에 복사되어 번들러가 `@a2ui/react/styles`로 접근할 수 있도록 한다.

## 의존성

### 외부 패키지
없음.

### 저장소 내부 모듈
- `../v0_8/styles/index` — v0_8 전용 스타일 진입점 (해당 파일의 문서는 현재 대상 목록에 없음)

## Exports

`export * from '../v0_8/styles/index'` 구문을 통해 `v0_8/styles/index`가 내보내는 모든 심벌을 재수출한다.

## 동작 흐름

단일 `export *` 구문만 존재한다. 스타일 관련 실제 로직은 `v0_8/styles/index`에 위임된다.
