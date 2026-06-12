# renderers/react/tests/v0_8/unit/components/Text.test.tsx

## 개요

v0.8 React 렌더러의 `Text` 컴포넌트에 대한 단위 테스트 파일이다. 기본 렌더링, 사용 힌트(usageHint), 테마 지원, 마크다운 렌더링, 보안(XSS 방어)을 총망라하여 검증한다. `vitest` + `@testing-library/react` 조합으로 실제 DOM 출력을 assertions한다.

## 의존성

### 외부 패키지
- `vitest`: `describe`, `it`, `expect`
- `@testing-library/react`: `render`, `screen`, `waitFor`
- `react`: `React`

### 저장소 내부 모듈
- [`../../utils`](../../utils/index.ts.md) — `TestWrapper`, `TestRenderer`, `createSimpleMessages`
- `../../../../src/v0_8` — `litTheme`, `defaultTheme` (테마 객체)

## Exports

없음 (테스트 파일, 전체가 side-effect인 `describe` 블록만 포함).

## 테스트 케이스 명세

모든 테스트는 `createSimpleMessages(id, componentType, props)` 헬퍼로 `Text` 컴포넌트 메시지를 생성한 뒤, `<TestWrapper><TestRenderer messages={...} /></TestWrapper>` 패턴으로 렌더링한다.

### `describe('Text Component')`

#### `describe('Basic Rendering')`

| 테스트 케이스 | 검증 동작 | 픽스처/모킹 |
|---|---|---|
| `should render text with literal string` | `'Hello World'` 텍스트 노드가 DOM에 존재함 | `text.literalString='Hello World'`, `usageHint='body'` |
| `should render text with whitespace only` | `.a2ui-surface` 엘리먼트가 존재함 (공백 문자열에도 surface가 렌더됨) | `text.literalString='   '`, `usageHint='body'` |
| `should render empty string` | `.a2ui-surface` 엘리먼트가 존재함 | `text.literalString=''`, `usageHint='body'` |

#### `describe('Usage Hints')`

`usageHints` 배열(`['h1','h2','h3','h4','h5','caption','body']`)을 순회하며 forEach로 동적 생성되는 테스트:

| 테스트 케이스 | 검증 동작 |
|---|---|
| `should render with usageHint="${hint}"` (7개) | `${hint} text` 텍스트 노드가 DOM에 존재함 |

추가 static 케이스 4개 — 모두 wrapper가 `<section>` 태그임을 검증 (Lit 렌더러 DOM 구조와 일치 목적, `usageHint`는 CSS 클래스에만 영향을 줌):

| 테스트 케이스 | 검증 동작 |
|---|---|
| `should render h1 with section wrapper` | `section` 엘리먼트 존재 및 textContent가 `'Main Title'`임 |
| `should render h2 with section wrapper` | `section` 엘리먼트 존재 및 textContent가 `'Section Title'`임 |
| `should render caption with section wrapper` | `section` 엘리먼트 존재 및 textContent가 `'Caption text'`임 |
| `should render body with section wrapper` | `section` 엘리먼트 존재 및 textContent가 `'Body text'`임 |

#### `describe('Theme Support')`

| 테스트 케이스 | 검증 동작 | 사용 테마 |
|---|---|---|
| `should apply default theme classes` | `section` 엘리먼트에 `layout-w-100` 클래스 존재 | `defaultTheme` (= litTheme) |
| `should apply lit theme classes for h1` | `section` 엘리먼트에 `typography-sz-hs` 클래스 존재 (litTheme의 `components.Text.h1` 설정) | `litTheme`, `usageHint='h1'` |
| `should apply body variant classes from lit theme` | `section` 엘리먼트에 `layout-w-100` 클래스 존재 (litTheme `all` 상속) | `litTheme`, `usageHint='body'` |

#### `describe('Markdown Rendering')`

각 케이스는 `text.literalString`에 마크다운 문법을 전달하고, 렌더된 DOM에서 대응하는 HTML 태그를 검증한다.

| 테스트 케이스 | 입력 | 검증 |
|---|---|---|
| `should render bold text` | `'This is **bold** text'` | `<strong>` 엘리먼트 존재, textContent가 `'bold'` |
| `should render italic text` | `'This is *italic* text'` | `<em>` 엘리먼트 존재, textContent가 `'italic'` |
| `should render inline code` | `` 'Use the `console.log()` function' `` | `<code>` 엘리먼트 존재, textContent가 `'console.log()'` |
| `should render unordered lists` | `'- Item 1\n- Item 2\n- Item 3'` | `<ul>` 존재, `<li>` 엘리먼트 3개 |
| `should render ordered lists` | `'1. First\n2. Second\n3. Third'` | `<ol>` 존재, `<li>` 엘리먼트 3개 |
| `should render links` | `'Visit [Google](https://google.com)'` | `<a>` 존재, `href='https://google.com'`, textContent `'Google'` |
| `should render plain URLs as text` | `'Check out https://example.com for more'` | container.textContent에 `'https://example.com'` 포함 (auto-linkify 미사용) |
| `should render blockquotes` | `'> This is a quote'` | `<blockquote>` 엘리먼트 존재 |
| `should render code blocks` | `` '```\nconst x = 1;\n```' `` | `<pre>` 존재, 그 안에 `<code>` 존재 |
| `should preserve line breaks in text` | `'Line 1\nLine 2'` | container.textContent에 `'Line 1'`, `'Line 2'` 포함 |

#### `describe('Markdown Theme Classes')`

| 테스트 케이스 | 검증 동작 |
|---|---|
| `should apply theme classes to markdown elements` | `<ul>` 엘리먼트에 `typography-f-s` 클래스 존재 (litTheme의 `markdown.ul`) |
| `should apply paragraph classes from theme` | `<p>` 엘리먼트에 `typography-sz-bm` 클래스 존재 (litTheme의 `markdown.p`) |

#### `describe('Security')`

| 테스트 케이스 | 입력 | 검증 |
|---|---|---|
| `should not render raw HTML` | `'<script>alert("xss")</script>'` | `<script>` 태그 없음, container.textContent에 이스케이프된 `'<script>'` 문자열 포함 |
| `should not render onclick handlers` | `'<div onclick="alert(1)">Click me</div>'` | `[onclick]` 속성을 가진 엘리먼트 없음 |
| `should not render iframe tags` | `'<iframe src="https://evil.com"></iframe>'` | `<iframe>` 엘리먼트 없음 |

## 동작 흐름

1. 각 테스트에서 `createSimpleMessages`로 `surfaceUpdate` + `beginRendering` 쌍의 메시지 배열을 생성한다.
2. `<TestWrapper>`가 `A2UIProvider`를 설정하고, `<TestRenderer>`가 `useEffect`에서 `processMessages`를 호출하여 상태를 초기화한다.
3. `waitFor` 또는 동기 assertions로 렌더된 DOM을 검증한다.
4. 테마가 필요한 테스트는 `<TestWrapper theme={litTheme|defaultTheme}>`로 테마를 주입한다.
