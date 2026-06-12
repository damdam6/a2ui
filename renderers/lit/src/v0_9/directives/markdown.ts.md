# renderers/lit/src/v0_9/directives/markdown.ts

## 개요

마크다운 문자열을 HTML로 변환하여 Lit 템플릿 안에 안전하게 렌더링하는 커스텀 Lit 디렉티브를 정의한다. 이전 렌더링 결과와 값이 동일하면 `noChange`를 반환하는 최적화를 포함하며, 렌더러가 없을 경우 경고 로그를 한 번만 출력하는 정적 플래그를 사용한다.

## 의존성

### 외부 패키지
- `lit`: `html`, `noChange`
- `lit/directive.js`: `Directive`, `DirectiveParameters`, `Part`, `directive`
- `lit/directives/unsafe-html.js`: `unsafeHTML`
- `lit/directives/until.js`: `until`
- `@a2ui/web_core/types/types`: `Types` 네임스페이스 (타입 전용)

## Exports

- `markdown` (함수): `directive(MarkdownDirective)` 호출 결과 — Lit 디렉티브 팩토리 함수

## 상세 명세

### `MarkdownDirective` 클래스 (내부, 비공개)

`Directive`를 상속하는 클래스.

#### 인스턴스 필드
- `private lastValue: string | null = null` — 이전에 렌더링된 마크다운 문자열 캐시
- `private lastTagClassMap: string | null = null` — 이전 `markdownOptions.tagClassMap`의 JSON 직렬화 캐시

#### `static defaultMarkdownWarningLogged: boolean = false`
마크다운 렌더러 미설정 경고가 이미 콘솔에 출력되었는지 여부를 추적하는 정적 플래그. 경고가 반복 출력되지 않도록 한다.

#### `update(_part: Part, [value, markdownRenderer, markdownOptions]: DirectiveParameters<this>): typeof noChange | TemplateResult`
- 시그니처: `update(_part: Part, [value, markdownRenderer, markdownOptions]: DirectiveParameters<this>)`
- `markdownOptions?.tagClassMap`을 `JSON.stringify`하여 `jsonTagClassMap`을 구한다.
- `this.lastValue === value`이고 `jsonTagClassMap === this.lastTagClassMap`이면 `noChange`를 반환하여 불필요한 재렌더링을 방지한다.
- 캐시 값이 달라졌으면 `this.lastValue`와 `this.lastTagClassMap`을 업데이트하고 `this.render(...)`를 호출한다.

#### `render(value: string, markdownRenderer?: Types.MarkdownRenderer, markdownOptions?: Types.MarkdownRendererOptions): TemplateResult`
- 시그니처: `render(value: string, markdownRenderer?: Types.MarkdownRenderer, markdownOptions?: Types.MarkdownRendererOptions)`
- **마크다운 렌더러가 있는 경우**: `markdownRenderer(value, markdownOptions)`를 호출하여 `Promise<string>`을 얻는다. Promise가 resolve되면 결과 문자열에 `unsafeHTML()` 디렉티브를 적용한다(HTML 이스케이프 없이 주입 — 렌더러가 직접 살균해야 함). Promise가 아직 pending인 동안 폴백으로 `html`<span class="no-markdown-renderer">${value}</span>`` 를 표시하기 위해 `until(rendered, placeholder)` 패턴을 사용한다.
- **마크다운 렌더러가 없는 경우**: `MarkdownDirective.defaultMarkdownWarningLogged`가 `false`이면 `console.warn`으로 "[MarkdownDirective] can't render markdown because no markdown renderer is configured." 메시지를 출력하고 플래그를 `true`로 설정한다. 이후 `html`<span class="no-markdown-renderer">${value}</span>``를 반환한다.

### `markdown` 상수 (exported)

- `directive(MarkdownDirective)` 호출 결과.
- Lit 템플릿에서 `markdown(value, renderer, options)` 형태로 사용한다.

## 동작 흐름

1. 소비자(예: `Text` 컴포넌트)가 `markdown(value, renderer, options)`를 Lit 템플릿 바인딩에 사용한다.
2. Lit가 파트를 업데이트할 때 `update()`를 호출한다.
3. 캐시 비교 후 값이 변경된 경우에만 `render()`가 실행된다.
4. `render()`는 렌더러 유무에 따라 비동기 HTML 삽입 경로 또는 플레인 텍스트 폴백 경로를 선택한다.
5. 비동기 경로에서는 `until` 디렉티브가 로딩 중 상태와 완료 상태 전환을 관리한다.
