# renderers/angular/a2ui_explorer/src/app/tests/v0_8/2_row_layout.spec.ts

## 개요

v0.8 버전의 "Row Layout (minimal)" 예제가 Angular 렌더러에서 올바르게 렌더링되는지 검증하는 Jasmine 테스트 스위트다. `loadExample`로 예제를 로드한 뒤 캔버스 텍스트 콘텐츠를 검사해 좌우 콘텐츠 영역 레이블(`'Left Content'`, `'Right Content'`)이 정상 표시되는지 확인한다. `ComponentFixture`를 직접 사용하지 않는 최소 구조의 렌더링 스모크 테스트다.

## 의존성

### 외부 패키지
없음 (Jasmine은 테스트 런타임으로 암묵적 제공)

### 저장소 내부 모듈
- [`../utils/test_utils`](../utils/test_utils.ts.md) — `Version`, `getCanvas`, `loadExample` 유틸리티 함수 제공

## Exports

이 파일은 Jasmine `describe` 블록을 최상위에 선언하며, TypeScript 모듈 export는 없다.

## 테스트 케이스 상세 명세

### `describe`: `'Example: Row Layout (minimal) (v0.8)'`

**픽스처 / 셋업**
- `textContent: string` — 각 테스트 전에 캔버스 DOM의 텍스트 전체를 저장하는 변수.
- `beforeEach`: `loadExample('Row Layout (minimal)', Version.V0_8)`을 `await`로 호출해 예제를 로드한 뒤, `getCanvas().textContent`로 캔버스 텍스트를 읽어 `textContent`에 저장한다.

---

#### 테스트 1: `'should render expected text content'`

| 검증 항목 | 기대 값 |
|-----------|---------|
| 좌측 콘텐츠 레이블 | `'Left Content'` |
| 우측 콘텐츠 레이블 | `'Right Content'` |

`textContent.toContain(...)` 방식으로 2개의 필수 문자열이 캔버스에 포함되는지 확인한다.

## 동작 흐름

`beforeEach` → `loadExample`로 예제 컴포넌트 초기화 → `getCanvas()` DOM 노드의 `textContent` 읽기 → 단일 테스트에서 좌/우 콘텐츠 레이블 존재 여부 검증.
