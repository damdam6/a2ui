# renderers/react/tests/v0_8/unit/components/SvgReset.test.tsx

## 개요

CSS 리셋 스타일에서 SVG 엘리먼트를 올바르게 제외하는지 검증하는 단위 테스트 파일이다. `resetStyles` 문자열에 올바른 CSS 선택자가 포함되어 있는지와, SVG 엘리먼트에 presentational attribute(`fill` 등)가 CSS 리셋에 의해 덮어쓰이지 않고 유지되는지 두 가지 측면을 확인한다.

## 의존성

### 외부 패키지
- `vitest` — `describe`, `it`, `expect`
- `@testing-library/react` — `render`
- `react` — `React`

### 저장소 내부 모듈
- [`../../../../src/v0_8`](../../../../src/v0_8/index.ts.md) — `A2UIProvider`
- [`../../../../src/v0_8/styles/reset`](../../../../src/v0_8/styles/reset.ts.md) — `resetStyles`

## Exports

이 파일은 아무것도 export하지 않는다. Vitest 테스트 스위트 파일이다.

## 상세 명세 — 테스트 케이스

### `describe('SVG Reset Exclusion')`

#### `should have the correct CSS selector in resetStyles`
- `resetStyles` 문자열(CSS 텍스트)에 `:not(svg, svg *:not(foreignObject *))` 선택자가 포함되어 있음을 `expect(resetStyles).toContain(...)` 로 검증한다.
- 이 선택자는 CSS 리셋 규칙이 SVG 및 SVG 자식 엘리먼트(단, `foreignObject` 내부는 제외)에는 적용되지 않도록 보장한다.
- 픽스처 없음, 모킹 없음, 순수 문자열 단언이다.

#### `should allow presentational attributes on SVG elements`
- `A2UIProvider`로 감싼 `div.a2ui-surface` 내부에 `fill="red"` 속성을 가진 `svg` 엘리먼트를 직접 JSX로 작성해 렌더링한다. SVG 내부에는 `circle` 엘리먼트가 포함된다.
- `container.querySelector('svg')`가 DOM에 존재하고, `fill` 속성값이 `'red'`임을 검증한다.
- 이를 통해 CSS 리셋이 SVG의 presentational attribute를 덮어쓰지 않음을 확인한다.

## 동작 흐름

1. 첫 번째 테스트는 `resetStyles` 모듈 상수를 직접 읽어 문자열 내용을 단언한다(렌더링 불필요).
2. 두 번째 테스트는 `A2UIProvider`(전역 스타일 주입 포함)와 함께 SVG를 렌더링한 뒤, 브라우저 DOM에서 `fill` 속성이 그대로 남아있는지 확인한다.
3. 두 테스트 모두 별도의 모킹 없이 실제 모듈과 실제 DOM을 사용한다.
