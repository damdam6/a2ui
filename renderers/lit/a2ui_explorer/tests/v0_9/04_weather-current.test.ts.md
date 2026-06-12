# renderers/lit/a2ui_explorer/tests/v0_9/04_weather-current.test.ts

## 개요

A2UI 예제 `04_weather-current.json`을 `<local-gallery>`에 로드하여 현재 날씨 카드 UI가 올바르게 렌더링되는지 검증하는 통합 테스트 파일이다. 지역명, 날씨 설명, 현재 기온, 이모지 아이콘, 예보 기온 수치, 그리고 예보 요일 이름의 렌더링 여부를 확인한다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈
- [`../utils/test-utils`](../utils/test-utils.ts.md) — `loadExample`, `getSurface`, `getDeepTextContent`
- [`../../src/local-gallery`](../../src/local-gallery.ts.md) — `LocalGallery` 타입

## Exports

없음 (테스트 파일)

## 테스트 케이스 명세

### `describe('Example: Weather Current')`

**픽스처 설정:**
- `gallery: LocalGallery`, `surface: HTMLElement`, `textContent: string`을 스위트 범위 변수로 선언.
- `beforeEach`: `loadExample('04_weather-current.json')`으로 갤러리를 마운트하고, `getSurface(gallery)`로 `a2ui-surface` 엘리먼트를 획득한 뒤, `getDeepTextContent(surface)`를 `textContent`에 저장.
- `afterEach`: `gallery?.remove()`로 DOM에서 갤러리를 제거.

---

**`it('should render main content')`**
- 검증 동작: 지역명, 날씨 설명 문자열, 고온·저온 기온이 렌더링된다.
- 기대 결과: `textContent`에 `'Austin, TX'`, `'Clear skies with light breeze'`, `'72°'`, `'58°'` 포함.

**`it('should render icons')`**
- 검증 동작: 날씨 아이콘으로 이모지 `☀️`와 `⛅`가 렌더링된다.
- 기대 결과: `textContent`에 `'☀️'`, `'⛅'` 포함.

**`it('should render forecast temperatures')`**
- 검증 동작: 5일 예보의 각 기온 수치가 렌더링된다.
- 기대 결과: `textContent`에 `'74°'`, `'76°'`, `'71°'`, `'73°'`, `'75°'` 포함.

**`it('should render day names for forecast')`**
- 검증 동작: 예보 섹션에 요일 약칭이 렌더링된다.
- 경계 케이스: 테스트 실행 시점에 따라 표시되는 요일이 달라지므로, `'Tue'`, `'Wed'`, `'Thu'`, `'Fri'`, `'Sat'` 중 하나 이상이 포함되면 통과한다. `withContext('Should render day names for forecast')`으로 실패 시 컨텍스트를 제공한다.

## 동작 흐름

모든 테스트는 `beforeEach`에서 준비된 `textContent`를 사용하는 정적 렌더링 검증이다. 예보 요일 이름은 실행 시점에 따라 달라질 수 있으므로 OR 조건으로 허용 범위를 넓혀 처리한다.
