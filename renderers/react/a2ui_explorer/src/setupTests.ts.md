# renderers/react/a2ui_explorer/src/setupTests.ts

## 개요

Vitest 기반 단위 테스트 환경 설정 파일이다. `@testing-library/jest-dom`을 import하여 `toBeInTheDocument()` 등 jest-dom 커스텀 매처를 전역으로 등록한다. `vite.config.ts`의 `test.setupFiles`에 지정되어 각 테스트 파일 실행 전에 자동으로 로드된다.

## 의존성

### 외부 패키지
- `@testing-library/jest-dom` — jest-dom 커스텀 매처 전역 등록 (사이드 이펙트 import)

### 저장소 내부 모듈
없음

## Exports

없음

## 동작 흐름

import 문 하나만 존재하며, 모듈 로드 시점에 jest-dom 매처가 전역 `expect`에 등록된다.
