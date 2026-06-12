# renderers/angular/a2ui_explorer/src/app/tests/v0_9/27_stats-card.spec.ts

## 개요

`Stats Card` 예제 컴포넌트에 대한 가장 단순한 형태의 Angular 통합 테스트 파일이다. `ComponentFixture`를 사용하지 않고, 텍스트와 아이콘 이름을 단일 테스트 케이스에서 동시에 검증한다.

## 의존성

### 외부 패키지
없음.

### 저장소 내부 모듈
- [`../utils`](../utils/index.ts.md) — `getCanvas`, `loadExample`

## Exports

직접 export하는 심볼 없음. Jasmine `describe` 블록으로 테스트 스위트를 등록한다.

## 테스트 케이스 명세

### `describe('Example: Stats Card', ...)`

#### 픽스처 / 모킹

- `textContent`: `string` — `getCanvas().textContent`.
- `beforeEach`는 `async`로 `loadExample('Stats Card')`를 호출하고, 반환 픽스처를 무시한 채 `textContent`만 갱신한다.

#### 테스트 케이스

1. **`should render text content and icons`**
   - 검증 동작: 캔버스 텍스트에 Material 아이콘 이름 `'trending_up'`, 텍스트 `'Monthly Revenue'`, 아이콘 이름 `'arrow_upward'`가 포함되는지 확인한다.
   - 픽스처/모킹: `textContent`.

## 동작 흐름

`beforeEach`에서 `'Stats Card'` 예제를 로드하고 텍스트를 스냅샷한다. 단일 테스트에서 텍스트 노드로 렌더링된 아이콘 이름과 레이블을 한 번에 검증한다.
