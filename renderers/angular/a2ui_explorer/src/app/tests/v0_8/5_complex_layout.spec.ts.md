# renderers/angular/a2ui_explorer/src/app/tests/v0_8/5_complex_layout.spec.ts

## 개요

v0.8 버전의 "Complex Layout (minimal)" 예제가 Angular 렌더러에서 올바르게 렌더링되는지 검증하는 Jasmine 테스트 스위트다. 복잡한 레이아웃 구조에서 사용자 프로필 폼 제목(`'User Profile Form'`)과 안내 문구(`'Please fill out all fields.'`)가 캔버스에 표시되는지 단일 텍스트 검증 테스트로 확인한다. `ComponentFixture`를 직접 사용하지 않는 단순 렌더링 스모크 테스트다.

## 의존성

### 외부 패키지
없음 (Jasmine은 테스트 런타임으로 암묵적 제공)

### 저장소 내부 모듈
- [`../utils/test_utils`](../utils/test_utils.ts.md) — `Version`, `getCanvas`, `loadExample` 유틸리티 함수 제공

## Exports

이 파일은 Jasmine `describe` 블록을 최상위에 선언하며, TypeScript 모듈 export는 없다.

## 테스트 케이스 상세 명세

### `describe`: `'Example: Complex Layout (minimal) (v0.8)'`

**픽스처 / 셋업**
- `textContent: string` — 각 테스트 전에 캔버스 DOM의 텍스트를 저장하는 변수.
- `beforeEach`: `loadExample('Complex Layout (minimal)', Version.V0_8)`를 `await`로 호출한 뒤, `getCanvas().textContent`를 `textContent`에 저장한다.

---

#### 테스트 1: `'should render expected text content'`

| 검증 항목 | 기대 값 |
|-----------|---------|
| 폼 제목 | `'User Profile Form'` |
| 안내 문구 | `'Please fill out all fields.'` |

`textContent.toContain(...)` 방식으로 2개의 문자열 존재 여부를 검증한다.

## 동작 흐름

`beforeEach` → `loadExample`로 복잡한 레이아웃 예제 초기화 → `getCanvas().textContent` 읽기 → 단일 테스트에서 폼 제목과 안내 문구 존재 검증.
