# renderers/angular/a2ui_explorer/src/app/tests/v0_8/29_movie-card.spec.ts

## 개요

v0.8 버전의 "Movie Card (basic)" 예제가 Angular 렌더러에서 올바르게 렌더링되는지 검증하는 Jasmine 테스트 스위트다. `loadExample`로 예제를 로드한 뒤 캔버스 DOM의 텍스트 콘텐츠를 검사하여 영화 카드에 필수 데이터(제목, 연도, 장르, 평점, 상영 시간)가 표시되는지 확인한다. `ComponentFixture`를 직접 사용하지 않는 단순 렌더링 스모크 테스트로, 단일 `describe`와 단일 `it`으로 구성된다.

## 의존성

### 외부 패키지
없음 (Jasmine은 테스트 런타임으로 암묵적 제공)

### 저장소 내부 모듈
- [`../utils/test_utils`](../utils/test_utils.ts.md) — `Version`, `getCanvas`, `loadExample` 유틸리티 함수 제공

## Exports

이 파일은 Jasmine `describe` 블록을 최상위에 선언하며, TypeScript 모듈 export는 없다.

## 테스트 케이스 상세 명세

### `describe`: `'Example: Movie Card (basic) (v0.8)'`

**픽스처 / 셋업**
- `textContent: string` — 각 테스트 전에 캔버스 DOM의 텍스트 전체를 저장하는 변수.
- `beforeEach`: `loadExample('Movie Card (basic)', Version.V0_8)`을 `await`로 호출해 예제를 로드한 뒤, `getCanvas().textContent`로 캔버스 텍스트를 읽어 `textContent`에 저장한다. `ComponentFixture`는 캡처하지 않는다.

---

#### 테스트 1: `'should render expected text content'`

| 검증 항목 | 기대 값 |
|-----------|---------|
| 영화 제목 | `'Interstellar'` |
| 개봉 연도 | `'(2014)'` |
| 장르 목록 | `'Sci-Fi • Adventure • Drama'` |
| 평점 | `'8.7/10'` |
| 상영 시간 | `'2h 49min'` |

`textContent.toContain(...)` 방식으로 각 문자열이 캔버스에 포함되는지 확인한다. 총 5개의 `expect` 호출.

## 동작 흐름

`beforeEach` → `loadExample`로 예제 컴포넌트 초기화 → `getCanvas()` DOM 노드의 `textContent` 읽기 → 단일 테스트에서 5개의 필수 텍스트 항목 존재 여부 검증.
