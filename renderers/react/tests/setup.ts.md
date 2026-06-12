# renderers/react/tests/setup.ts

## 개요

Vitest 전역 테스트 셋업 파일로, 모든 테스트 스위트가 실행되기 전에 한 번 호출되는 공통 초기화 로직을 담당한다. `@testing-library/jest-dom`의 커스텀 매처를 Vitest 환경에 등록하고, A2UI의 기본 컴포넌트 카탈로그를 초기화한다. 이 파일은 vitest 설정의 `setupFiles` 항목에 지정되어 자동으로 실행되도록 구성된다.

## 의존성

### 외부 패키지
- `@testing-library/jest-dom/vitest` — `toBeInTheDocument()` 등 DOM 관련 커스텀 매처를 Vitest expect에 등록
- `vitest` — `beforeAll` 훅

### 저장소 내부 모듈
- [`../src`](../src/index.ts.md) — `initializeDefaultCatalog` 함수 제공 (React 렌더러 메인 진입점)

## Exports

이 파일은 아무것도 export하지 않는다. 사이드 이펙트 전용 셋업 파일이다.

## 상세 명세

### `beforeAll` 훅 (모듈 레벨 등록)

- **시그니처**: `beforeAll(() => void)`
- **동작**: Vitest가 테스트 스위트를 실행하기 직전 콜백을 한 번 호출한다. 콜백 내부에서 `initializeDefaultCatalog()`를 실행하여 A2UI 내장 컴포넌트(Button, Text, Card 등)를 전역 카탈로그에 등록한다. 이를 통해 개별 테스트 파일에서 별도 초기화 없이 모든 기본 컴포넌트를 사용할 수 있다.

## 동작 흐름

파일이 import되는 순간 `@testing-library/jest-dom/vitest`의 매처 확장이 등록된다. 이후 `beforeAll` 훅이 Vitest 런타임에 등록되고, 첫 번째 테스트 실행 직전에 `initializeDefaultCatalog()`가 호출된다.
