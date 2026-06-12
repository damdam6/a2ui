# renderers/angular/a2ui_explorer/src/app/tests/v0_9/04_weather-current.spec.ts

## 개요

v0.9 버전의 "Weather Current" 예제가 Angular 렌더러에서 올바르게 렌더링되는지 검증하는 Jasmine 테스트 스위트다. 현재 날씨 정보(도시명, 날씨 설명, 기온), 이모지 아이콘, 5일 예보 기온, 그리고 예보 요일 약어를 총 4개의 테스트로 검증한다. 인터랙션 없이 순수 렌더링만 확인한다.

## 의존성

### 외부 패키지
없음 (Jasmine은 테스트 런타임으로 암묵적 제공)

### 저장소 내부 모듈
- [`../utils`](../utils/index.ts.md) — `getCanvas`, `loadExample` 유틸리티 함수 제공

## Exports

이 파일은 Jasmine `describe` 블록을 최상위에 선언하며, TypeScript 모듈 export는 없다.

## 테스트 케이스 상세 명세

### `describe`: `'Example: Weather Current'`

**픽스처 / 셋업**
- `textContent: string` — 각 테스트 전에 캔버스 DOM의 텍스트를 저장하는 변수.
- `beforeEach`: `loadExample('Weather Current')`를 `await`로 호출한 뒤, `getCanvas().textContent`를 저장한다.

---

#### 테스트 1: `'should render main content'`

캔버스 텍스트에 다음이 포함되는지 확인한다.
- 도시명: `'Austin, TX'`
- 날씨 설명: `'Clear skies with light breeze'`
- 최고/최저 기온: `'72°'`, `'58°'`

---

#### 테스트 2: `'should render icons'`

캔버스 텍스트에 날씨 이모지 아이콘이 포함되는지 확인한다.
- 맑은 날씨: `'☀️'`
- 구름 많음: `'⛅'`

---

#### 테스트 3: `'should render forecast temperatures'`

5일 예보의 각 기온 값이 캔버스에 포함되는지 확인한다.
- `'74°'`, `'76°'`, `'71°'`, `'73°'`, `'75°'`

---

#### 테스트 4: `'should render day names for forecast'`

예보 요일 약어 중 하나 이상이 포함되는지 OR 조건으로 확인한다. `'Tue'`, `'Wed'`, `'Thu'`, `'Fri'`, `'Sat'` 중 하나가 `true`이면 `hasDayName`이 `true`로 평가되며, `withContext('Should render day names for forecast')`와 함께 `toBeTruthy()`를 검증한다.

**경계 케이스**: 요일 약어는 실행 시점의 날짜에 따라 달라질 수 있어 여러 후보를 OR로 허용한다.

## 동작 흐름

`beforeEach` → `loadExample`로 예제 초기화 → 텍스트 스냅샷 → 테스트 1(현재 날씨 텍스트) → 테스트 2(날씨 이모지) → 테스트 3(예보 기온 5개) → 테스트 4(동적 요일 약어 OR 검증).
