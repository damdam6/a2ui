# renderers/angular/a2ui_explorer/src/app/tests/v0_8/27_stats-card.spec.ts

## 개요

v0.8 카탈로그의 "Stats Card (basic)" 예제가 Angular 렌더러에서 올바르게 렌더링되는지 검증하는 Jasmine 테스트 파일이다. 매출 통계 카드에 표시되어야 하는 아이콘, 레이블, 수치, 증감율 텍스트를 단일 테스트 케이스로 확인한다. 인터랙션 테스트 없이 텍스트 렌더링 검증만 수행한다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈
- [`../utils/test_utils`](../utils/test_utils.ts.md) — `Version`, `getCanvas`, `loadExample`

## Exports

없음 (테스트 파일이므로 export 없음)

## 테스트 케이스 명세

### `describe`: `'Example: Stats Card (basic) (v0.8)'`

#### 픽스처 / 설정
- `textContent: string` — 각 테스트 실행 전 캔버스 요소의 텍스트 내용
- `beforeEach`: `loadExample('Stats Card (basic)', Version.V0_8)`를 `await`하여 예제를 로드한 후, `getCanvas().textContent`를 `textContent`에 저장한다.

#### 테스트 케이스: `'should render expected text content'`
- **검증 동작**: 캔버스 텍스트에 다음 항목이 모두 포함되어 있는지 `toContain`으로 단언한다.
  - 주요 아이콘 텍스트: `'trending_up'`
  - 카드 제목: `'Monthly Revenue'`
  - 수치: `'$48,294'`
  - 변화 방향 아이콘 텍스트: `'arrow_upward'`
  - 증감율 문구: `'+12.5% from last month'`
- **사용 픽스처/모킹**: `textContent`, `loadExample`, `getCanvas`

## 동작 흐름

1. `beforeEach`에서 `loadExample`을 호출해 Stats Card 예제를 로드한다.
2. `getCanvas().textContent`로 렌더링된 캔버스 DOM 전체 텍스트를 추출한다.
3. 단일 `it` 블록에서 통계 카드의 아이콘 텍스트, 제목, 금액, 증감율이 모두 포함되는지 검증한다. 아이콘 검증을 `textContent`로 하는 점에서 Material Icons의 ligature 방식 렌더링에 의존함을 알 수 있다.
