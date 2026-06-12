# renderers/angular/a2ui_explorer/src/app/tests/v0_8/23_step-counter.spec.ts

## 개요

v0.8 카탈로그의 "Step Counter (basic)" 예제가 Angular 렌더러에서 올바르게 렌더링되는지 검증하는 Jasmine 테스트 파일이다. 걸음 수, 진행률, 거리, 칼로리 등의 피트니스 통계 텍스트를 단일 테스트 케이스로 확인한다. 인터랙션 테스트 없이 텍스트 렌더링 검증만 수행한다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈
- [`../utils/test_utils`](../utils/test_utils.ts.md) — `Version`, `getCanvas`, `loadExample`

## Exports

없음 (테스트 파일이므로 export 없음)

## 테스트 케이스 명세

### `describe`: `'Example: Step Counter (basic) (v0.8)'`

#### 픽스처 / 설정
- `textContent: string` — 각 테스트 실행 전 캔버스 요소의 텍스트 내용
- `beforeEach`: `loadExample('Step Counter (basic)', Version.V0_8)`를 `await`하여 예제를 로드한 후, `getCanvas().textContent`를 `textContent`에 저장한다.

#### 테스트 케이스: `'should render expected text content'`
- **검증 동작**: 캔버스 텍스트에 다음 항목이 모두 포함되어 있는지 `toContain`으로 단언한다.
  - 섹션 제목: `"Today's Steps"`
  - 통계 레이블: `'Distance'`, `'Calories'`
  - 걸음 수: `'8,432'`
  - 진행률 텍스트: `'84% of 10,000 goal'`
  - 거리: `'3.8 mi'`
  - 칼로리: `'312'`
- **사용 픽스처/모킹**: `textContent`, `loadExample`, `getCanvas`

## 동작 흐름

1. `beforeEach`에서 `loadExample`을 호출해 Step Counter 예제를 로드한다.
2. `getCanvas().textContent`로 렌더링된 캔버스 DOM 전체 텍스트를 추출한다.
3. 단일 `it` 블록에서 걸음 수 위젯의 모든 수치 및 레이블 텍스트가 포함되는지 검증한다.
