# renderers/lit/src/v0_9/tests/markdown.test.ts

## 개요

Lit `markdown` 디렉티브의 두 가지 동작을 검증하는 단위 테스트 파일이다. 마크다운 렌더러가 제공되지 않았을 때 폴백(fallback) 스팬을 렌더링하는지, 그리고 비동기 `MarkdownRenderer`가 제공되었을 때 프로미스가 resolve된 뒤 실제 HTML로 교체되는지를 확인한다.

## 의존성

### 외부 패키지
- `node:assert` — `assert`
- `node:test` — `describe`, `it`, `before`, `after`
- `@a2ui/web_core/types/types` — `Types` (네임스페이스 import, `MarkdownRenderer` 타입 사용)
- `lit` — `html`, `render` (동적 import)

### 저장소 내부 모듈
- [`./dom-setup.js`](./dom-setup.ts.md) — `setupTestDom`, `teardownTestDom`, `asyncUpdate`
- `../directives/markdown.js` — `markdown` 디렉티브 (동적 import)

## Exports

없음 (테스트 파일)

## 테스트 케이스

### describe: `Markdown Directive`

**픽스처 / 모킹**
- `before` 훅: `setupTestDom()`을 호출하여 JSDOM 전역을 설정한다. lit 관련 import 이전에 반드시 호출해야 한다.
- `after` 훅: `teardownTestDom()`을 호출한다.
- 비고: `lit`와 `../directives/markdown.js`는 각 `it` 블록 내부에서 동적으로 import된다. 이는 JSDOM이 활성화된 후에 Lit 모듈이 평가되도록 보장하기 위함이다.

---

#### `should render fallback when no renderer is provided`
- 검증 동작:
  1. `lit`의 `html`과 `render`, `../directives/markdown.js`의 `markdown`을 동적으로 import한다.
  2. `document.createElement('div')`로 컨테이너를 생성한다.
  3. `render(html\`<div>${markdown('Hello world')}</div>\`, container)`를 실행한다. 렌더러 인수 없이 호출한다.
  4. `container.innerHTML`에 `'no-markdown-renderer'` 문자열이 포함되어 있음을 검증한다 (폴백 스팬 클래스).
  5. `container.textContent`에 `'Hello world'`가 포함되어 있음을 검증한다 (폴백 텍스트 렌더링).

#### `should render parsed markdown when renderer is provided`
- 검증 동작:
  1. `lit`과 `markdown` 디렉티브를 동적 import한다.
  2. 컨테이너 `div`를 생성한다.
  3. `Promise<string>`의 `resolve` 함수를 외부 변수 `resolveRenderer`로 노출하는 프로미스 `renderPromise`를 생성한다. 이를 통해 나중에 테스트에서 프로미스를 수동으로 resolve할 수 있다.
  4. `async () => renderPromise` 시그니처를 가진 `mockRenderer: Types.MarkdownRenderer`를 생성한다.
  5. `render(html\`<div>${markdown('Hello markdown', mockRenderer)}</div>\`, container)`를 실행한다.
  6. 프로미스가 아직 pending인 상태에서 `container.innerHTML`에 `'no-markdown-renderer'`가 포함되어 있음을 검증한다 (프로미스 해결 전 플레이스홀더 표시 확인).
  7. `asyncUpdate(container, () => { resolveRenderer!('<b>Rendered HTML</b>'); })`를 호출하여 매크로태스크 큐를 통해 프로미스를 resolve하고 렌더링이 완료되기를 기다린다.
  8. `container.innerHTML`에 `'<b>Rendered HTML</b>'`가 포함되어 있음을 검증한다.
  9. `container.innerHTML`에 `'no-markdown-renderer'`가 더 이상 없음을 검증한다.

## 동작 흐름

JSDOM 전역 설정 후 각 테스트 내에서 Lit와 `markdown` 디렉티브를 동적으로 로드한다. 첫 번째 테스트는 렌더러 없는 동기적 폴백 경로를 검증하고, 두 번째 테스트는 비동기 렌더러의 프로미스 pending → resolved 전환과 DOM 교체 동작을 검증한다. `resolveRenderer` 변수를 통해 프로미스 제어권을 테스트 코드로 노출하는 패턴을 사용한다.
