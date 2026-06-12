# renderers/react/visual-parity/tests/visual-parity.spec.ts

## 개요

Playwright 기반의 비주얼 패리티(Visual Parity) 통합 테스트 파일이다. Lit 렌더러를 기준(reference)으로 삼아 동일한 픽스처를 React 렌더러로 렌더링한 뒤, 두 스크린샷을 픽셀 단위로 비교하여 두 구현체의 시각적 일관성을 검증한다. 기준 스냅샷 파일 없이 라이브 비교만 수행하며, 여러 테마에 걸쳐 반복 검증한다.

## 의존성

### 외부 패키지
- `@playwright/test` — `test`, `expect`
- `pngjs` — `PNG`
- `pixelmatch` — 픽셀 차이 비교 라이브러리

### 저장소 내부 모듈
- [`../fixtures`](../fixtures/index.ts.md) — `allFixtures`, `fixtureNames`, `FixtureName`
- [`../fixtures/themes`](../fixtures/themes/index.ts.md) — `themeNames`, `ThemeName`

## 상수

| 이름 | 값 | 설명 |
|---|---|---|
| `REACT_BASE_URL` | `'http://localhost:5001'` | React 개발 서버 URL |
| `LIT_BASE_URL` | `'http://localhost:5002'` | Lit 개발 서버 URL |
| `PIXEL_DIFF_THRESHOLD` | `0.01` | pixelmatch에 전달하는 픽셀당 색상 차이 허용 임계값 (1%) |
| `MAX_DIFF_PERCENT` | `1` | 전체 픽셀 중 다를 수 있는 최대 비율(%) |
| `skippedFixtures` | `['dateTimeInputTime', 'multipleChoice', 'videoBasic', 'videoWithPathBinding']` | 비주얼 패리티 테스트에서 제외되는 픽스처 이름 목록 |
| `fixturesToTest` | `fixtureNames.filter(name => !skippedFixtures.includes(name))` | 실제 테스트 대상 픽스처 배열 |
| `themesToTest` | `['lit', 'visualParity', 'minimal']` | 테스트 대상 테마 이름 배열 (`'default'` 제외 — Lit에서 크래시 발생) |

## 헬퍼 함수

### `compareImages(img1Buffer: Buffer, img2Buffer: Buffer): {diffPixels: number; totalPixels: number; diffPercent: number}`

두 PNG 버퍼를 읽어 픽셀 차이를 계산하는 순수 함수다.

1. `PNG.sync.read`로 두 버퍼를 파싱한다.
2. 두 이미지의 너비 또는 높이가 다를 경우 `diffPixels: -1`, `diffPercent: 100`을 즉시 반환한다(크기 불일치 = 100% 다름).
3. `totalPixels = img1.width * img1.height`를 계산한다.
4. `pixelmatch`를 `threshold: PIXEL_DIFF_THRESHOLD`로 호출하고, 세 번째 인자(diff 이미지 버퍼)는 `null`로 전달하여 diff 이미지를 생성하지 않는다.
5. `diffPercent = (diffPixels / totalPixels) * 100`을 계산하여 반환한다.

### `buildUrl(baseUrl: string, fixture: string, theme?: string): string`

베이스 URL에 `?fixture=<name>` 과 선택적으로 `&theme=<name>` 파라미터를 붙여 반환한다. `theme`이 `'default'`이거나 없으면 `theme` 파라미터를 추가하지 않는다.

## 테스트 케이스

### `test.describe('Visual Parity: React vs Lit')`

`themesToTest` × `fixturesToTest` 조합에 대해 2중 루프로 테스트를 생성한다.

**테스트 구조:** `test.describe('Theme: ${theme}') > test('${fixture}')`

**검증하는 동작:**
1. `browser.newContext()`로 Lit용과 React용 두 개의 독립적인 브라우저 컨텍스트를 생성한다(상태 격리).
2. 각 컨텍스트에서 `newPage()`로 페이지를 만들고 `buildUrl`로 URL을 구성하여 이동한다.
3. `.fixture-container` 셀렉터가 `visible` 상태가 될 때까지, 그리고 `networkidle`까지 대기한다.
4. `page.evaluate(() => document.fonts.ready)`로 폰트 로드 완료를 기다린다(텍스트 렌더링 패리티 중요).
5. `.fixture-container` 로케이터를 스크린샷 찍는다.
6. `compareImages(reactScreenshot, litScreenshot)`로 픽셀 차이를 계산한다.
7. `diffPercent`를 콘솔에 `[${theme}] ${fixture}: ${diffPixels}/${totalPixels} pixels differ (N.NN%)` 형식으로 출력한다.
8. `expect(diffPercent).toBeLessThanOrEqual(MAX_DIFF_PERCENT)`로 1% 이내 차이인지 단언한다. 실패 시 메시지에 테마, 픽스처, 차이 값, 허용 최대치를 포함한다.
9. `finally` 블록에서 두 컨텍스트를 모두 `close()`한다.

**픽스처/모킹:** 별도 모킹 없음. 두 개발 서버(포트 5001, 5002)가 미리 실행 중이어야 한다.

---

### `test.describe('DOM Structure Debug')`

항상 `expect(true).toBe(true)`로 통과하는 디버깅용 테스트 3개를 포함한다.

#### `test('debug - compare dimensions')`
픽스처 `buttonPrimary`, 테마 `lit`에 대해 Lit(Shadow DOM 탐색)과 React(Light DOM 탐색) 양쪽의 컨테이너 치수, 버튼·텍스트 색상, CSS 변수값을 추출하여 콘솔에 JSON으로 출력한다.

- Lit 측: `themed-a2ui-surface → shadowRoot → a2ui-surface → shadowRoot → a2ui-button → shadowRoot → button` 경로로 접근하여 컴퓨티드 스타일과 `--p-100`, `--n-10`, `--n-90` CSS 변수를 수집한다.
- React 측: `.a2ui-surface button`, `.a2ui-surface section`, `.a2ui-surface p` 셀렉터로 접근한다. CSS 규칙 매칭 루프를 실행하여 `<p>` 에 적용 중인 color 관련 규칙을 최대 15개 수집한다. 임시 테스트 div를 생성하여 `color-c-n10` 클래스의 색상도 확인한다.

#### `test('debug - compare DOM')`
픽스처 `dividerHorizontal`, 테마 `lit`에 대해 Lit과 React의 DOM 구조와 컴퓨티드 스타일을 콘솔에 출력한다.

- 콘솔 `error` 이벤트와 `pageerror`를 `errors` 배열에 수집한다.
- Lit 측: `.fixture-container` innerHTML, `a2ui-surface` Shadow DOM, `a2ui-root` Shadow DOM, `a2ui-divider` Shadow DOM, `<hr>` 컴퓨티드 스타일(height, width, display, background)과 컨테이너 레이아웃 정보를 수집한다.
- React 측: `.a2ui-surface section`과 `.a2ui-surface h1`의 폰트, 색상, 마진 등 컴퓨티드 스타일을 수집한다.
- 수집한 정보를 레이블 구분선(`===`)과 함께 콘솔에 출력한다. 1000ms(Lit) 및 500ms(React) `waitForTimeout`을 사용한다.

#### `test('debug - list all fixtures')`
브라우저 없이 실행되는 순수 정보 출력 테스트다. `fixtureNames`의 전체 픽스처, 스킵 목록, 테스트 대상 수를 번호를 붙여 나열하고, `themesToTest`와 총 테스트 케이스 수(`fixturesToTest.length * themesToTest.length`)를 출력한다.

## 동작 흐름

1. 상수 정의 및 테스트 대상 픽스처/테마 배열 구성 (모듈 로드 시)
2. `test.describe` 블록이 Playwright 테스트 러너에 등록됨
3. 각 `(theme, fixture)` 조합에 대해: 두 브라우저 컨텍스트 생성 → 각각 URL 이동 → 렌더 완료 대기 → 스크린샷 → 픽셀 비교 → 임계치 단언
4. 컨텍스트를 항상 정리(`finally`)
