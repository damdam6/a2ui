# renderers/markdown/markdown-it/src/markdown.test.ts

## 개요

이 파일은 `MarkdownItRenderer` 클래스와 `renderMarkdown` 함수에 대한 Node.js 내장 테스트 러너(`node:test`) 기반 통합 테스트 파일이다. jsdom을 사용하여 Node.js 환경에서 DOMPurify가 필요로 하는 `window`와 `document`를 `globalThis`에 주입한 뒤 테스트를 실행한다. HTML 렌더링 정확성, tagClassMap 동작, XSS 방어, 무상태성을 검증한다.

## 의존성

### 저장소 내부 모듈

- [`./raw-markdown.js`](./raw-markdown.ts.md) — `MarkdownItRenderer` 클래스를 가져온다.
- [`./markdown.js`](./markdown.ts.md) — `renderMarkdown` 함수를 가져온다.

### 외부 패키지

- `jsdom` — Node.js 환경에서 DOM 환경을 시뮬레이션하는 패키지. DOMPurify 초기화에 필요한 `window`/`document` 제공.
- `node:test` — Node.js 내장 테스트 러너. `describe`, `it`을 제공.
- `node:assert` — Node.js 내장 어서션 모듈.

## 환경 설정

파일 최상단에서 `new JSDOM('')`으로 jsdom 인스턴스를 생성하고, `globalThis.window`와 `globalThis.document`를 각각 `jsdom.window`와 `jsdom.window.document`로 설정한다. 이는 DOMPurify의 Node.js 환경 초기화 요건을 충족시키기 위한 사전 설정이며, 모듈 import보다 앞서 실행되어야 하므로 파일 최상단에 배치된다.

## 테스트 케이스 명세

### `describe('MarkdownItRenderer')`

#### `it('renders basic markdown')`
- **검증 동작:** `new MarkdownItRenderer()`로 인스턴스 생성 후 `renderer.render('# Hello')`가 `/<h1>Hello<\/h1>/` 패턴에 일치하는 HTML을 반환하는지 확인한다.
- **픽스처/모킹:** 없음.

#### `it('applies tag classes via tagClassMap')`
- **검증 동작:** `render('# Hello', {h1: ['custom-class']})`가 `/<h1 class="custom-class">Hello<\/h1>/`에 일치하는 결과를 반환하는지 검증한다. tagClassMap의 단일 클래스 주입 동작을 확인한다.
- **픽스처/모킹:** 없음.

#### `it('applies multiple classes')`
- **검증 동작:** `render('para', {p: ['class1', 'class2']})`가 `/<p class="class1 class2">para<\/p>/`에 일치하는지 확인한다. 동일 태그에 복수 클래스가 공백으로 구분되어 결합되는 동작을 검증한다.
- **픽스처/모킹:** 없음.

#### `it('is stateless (tagClassMap does not persist)')`
- **검증 동작:** 동일 인스턴스로 두 번 연속 render를 호출할 때, 첫 번째 호출에서 전달한 tagClassMap이 두 번째 호출에 영향을 주지 않음을 검증한다. 첫 번째 결과(`result1`)에는 `class="persistent?"` 패턴이 존재해야 하고, 두 번째 결과(`result2`)에는 해당 패턴이 없어야 하며 `/<h1>Hello<\/h1>/`과 일치해야 한다.
- **픽스처/모킹:** 없음.

#### `it('handles empty tagClassMap')`
- **검증 동작:** `render('# Hello', {})`처럼 빈 객체를 tagClassMap으로 전달했을 때 `/<h1>Hello<\/h1>/` 패턴에 일치하는 기본 HTML이 반환되는지 확인한다.
- **픽스처/모킹:** 없음.

---

### `describe('renderMarkdown')`

#### `it('renders markdown successfully')`
- **검증 동작:** `renderMarkdown('# Hello World')`가 `/<h1>Hello World<\/h1>/`에 일치하는 HTML로 resolve되는지 확인한다. 기본 비동기 렌더링 흐름 전체를 검증한다.
- **픽스처/모킹:** 없음.

#### `it('sanitizes malicious markdown links')`
- **검증 동작:** `'This is a test [link](javascript:alert("XSS"))'`를 입력했을 때 결과 HTML이 `href="javascript:alert` 패턴을 포함하지 않아야 하고, 원본 마크다운 텍스트 `[link](javascript:alert("XSS"))`가 이스케이프된 형태로 남아 있어야 함을 검증한다. markdown-it이 javascript: 링크를 기본적으로 제거하고 DOMPurify가 2차 방어선 역할을 함을 확인한다.
- **픽스처/모킹:** 없음.

#### `it('safely escapes HTML input without enabling raw HTML')`
- **검증 동작:** `'This is a test <script>alert("XSS")</script>'` 입력에 대해 결과가 `&lt;script&gt;alert("XSS")&lt;\/script&gt;` 패턴에 일치하고, `<script>` 태그가 그대로 포함되지 않음을 검증한다. markdown-it의 raw HTML 비활성화 동작과 이스케이프 동작을 함께 확인한다.
- **픽스처/모킹:** 없음.

#### `it('preserves safe HTML output')`
- **검증 동작:** `'This is **bold** and *italic*.'`를 렌더링한 결과에 `/<strong>bold<\/strong>/`와 `/<em>italic<\/em>/` 패턴이 모두 존재하는지 확인한다. sanitize 과정에서 안전한 HTML 태그가 제거되지 않음을 검증한다.
- **픽스처/모킹:** 없음.

#### `it('preserves classnames applied via tagClassMap')`
- **검증 동작:** `'# Heading\n\nParagraph text'`를 `tagClassMap: {h1: ['text-h1', 'bold'], p: ['body-text']}`와 함께 렌더링할 때, 결과에 `/<h1 class="text-h1 bold">Heading<\/h1>/`와 `/<p class="body-text">Paragraph text<\/p>/`가 모두 존재하는지 확인한다. `renderMarkdown`의 tagClassMap 전달 경로 전체를 검증한다.
- **픽스처/모킹:** 없음.

## 동작 흐름

1. 파일 로드 시 jsdom으로 `globalThis.window`, `globalThis.document` 설정 → DOMPurify 초기화 준비 완료.
2. `describe('MarkdownItRenderer')` 블록: `MarkdownItRenderer` 인스턴스를 각 테스트마다 새로 생성하여 동기 `render` 메서드 동작을 검증.
3. `describe('renderMarkdown')` 블록: 비동기 `renderMarkdown` 함수를 호출하여 전체 렌더링 + sanitize 파이프라인을 검증.
