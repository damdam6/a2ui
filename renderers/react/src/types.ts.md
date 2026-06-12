# renderers/react/src/types.ts

## 개요

패키지 루트에서 타입을 임포트할 수 있는 진입점으로, `v0_8/types`의 모든 타입 심벌을 재수출한다. 사용자는 `@a2ui/react`의 공개 API 타입(컴포넌트 props, 훅 반환값, 설정 타입 등)을 이 경로를 통해 접근할 수 있다.

## 의존성

### 외부 패키지
없음.

### 저장소 내부 모듈
- [`./v0_8/types`](./v0_8/types.ts.md) — v0_8의 타입 정의 모음

## Exports

`export * from './v0_8/types'` 구문을 통해 `v0_8/types`가 내보내는 모든 타입 심벌을 재수출한다. 구체적인 타입 목록은 `v0_8/types.ts`를 참조한다.

## 동작 흐름

단일 `export *` 구문만 존재한다. 실제 타입 정의는 `v0_8/types.ts`에 위임된다.
