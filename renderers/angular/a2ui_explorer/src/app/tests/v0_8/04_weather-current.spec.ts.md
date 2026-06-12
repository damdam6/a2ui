# renderers/angular/a2ui_explorer/src/app/tests/v0_8/04_weather-current.spec.ts

## 개요

v0.8 프로토콜의 "Weather Current (basic)" 예제의 렌더링을 검증하는 Jasmine 통합 테스트 파일이다. 기온, 지역명, 날씨 설명 텍스트와 날씨 예보 이모지 아이콘이 올바르게 렌더링되는지 확인한다. 버튼 인터랙션은 없는 순수 렌더링 검증 파일이다.

## 의존성

### 저장소 내부 모듈
- [`../utils/test_utils`](../utils/test_utils.ts.md) — `Version`, `getCanvas`, `loadExample`

## Exports

없음 (spec 파일).

## 테스트 케이스

**describe**: `'Example: Weather Current (basic) (v0.8)'`

**공유 변수**: `textContent: string`

**`beforeEach` 설정**:
- `loadExample('Weather Current (basic)', Version.V0_8)`를 awaiting한다. fixture 반환값은 사용하지 않아 변수에 저장하지 않는다.
- `getCanvas().textContent`를 `textContent`에 저장한다.

**테스트 1: `'should render expected text content'`**
- 검증: `textContent`에 다음 내용이 모두 포함되는지 확인한다.
  - `'72°'` — 현재 기온
  - `'58°'` — 최저 기온
  - `'Austin, TX'` — 지역명
  - `'Clear skies with light breeze'` — 날씨 설명 문자열
  - `'☀️'` (`'☀️'`) — 맑음 예보 이모지 (주석: `// ☀️`)
  - `'⛅'` (`'⛅'`) — 구름 조금 예보 이모지 (주석: `// ⛅`)

**사용 픽스처/모킹**: `loadExample`이 TestBed를 구성하며 실제 v0.8 렌더러를 사용한다. fixture 객체를 직접 참조하지 않고 전역 DOM(`getCanvas()`)의 텍스트만 검사한다.
