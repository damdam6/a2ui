# renderers/angular/a2ui_explorer/src/app/tests/v0_9/28_countdown-timer.spec.ts

## 개요

`Countdown Timer` 예제 컴포넌트에 대한 Angular 통합 테스트 파일이다. 카운트다운 타이머 UI의 제목, 숫자(일/시/분), 단위 레이블, 목표 날짜 텍스트를 검증하는 2개의 테스트 케이스로 구성된다. `ComponentFixture`를 직접 사용하지 않는다.

## 의존성

### 외부 패키지
없음.

### 저장소 내부 모듈
- [`../utils`](../utils/index.ts.md) — `getCanvas`, `loadExample`

## Exports

직접 export하는 심볼 없음. Jasmine `describe` 블록으로 테스트 스위트를 등록한다.

## 테스트 케이스 명세

### `describe('Example: Countdown Timer', ...)`

#### 픽스처 / 모킹

- `textContent`: `string` — `getCanvas().textContent`.
- `beforeEach`는 `async`로, `loadExample('Countdown Timer')`를 호출하고 반환값을 무시한 채 `textContent`만 갱신한다.

#### 테스트 케이스

1. **`should render text content`**
   - 검증 동작: 캔버스 텍스트에 `'Product Launch'`, `'14'`, `'08'`, `'32'`, `'Days'`, `'Hours'`, `'Minutes'`가 포함되는지 확인한다. 각 숫자는 일, 시, 분 값을 나타낸다.
   - 픽스처/모킹: `textContent`.

2. **`should render date`**
   - 검증 동작: 캔버스 텍스트에 `'January'`와 `'2025'`가 포함되는지 확인한다. 타이머의 목표 날짜가 January 2025임을 검증한다.
   - 픽스처/모킹: `textContent`.

## 동작 흐름

`beforeEach`에서 `'Countdown Timer'` 예제를 로드하고 텍스트 스냅샷을 설정한다. 두 테스트는 각각 타이머 표시 값과 목표 날짜 문자열을 독립적으로 검증한다.
