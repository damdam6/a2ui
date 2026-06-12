# renderers/lit/src/v0_9/catalogs/basic/components/Text.ts

## 개요

`a2ui-basic-text` 커스텀 엘리먼트를 정의하는 파일이다. 텍스트를 마크다운으로 렌더링하며 `h1`~`h5`, `body`, `caption` 등 여러 variant를 지원한다. `@lit/context`로 주입된 외부 마크다운 렌더러를 사용하며, 없을 경우 내장 `markdown` 디렉티브로 폴백한다. `caption` variant는 CSS 클래스로, `h1`~`h5` variant는 마크다운 헤딩 접두어를 텍스트에 추가하는 방식으로 처리한다.

## 의존성

### 외부 패키지
- `lit` — `html`, `nothing`, `css`
- `lit/decorators.js` — `customElement`
- `@lit/context` — `consume`
- `@a2ui/web_core/v0_9/basic_catalog` — `TextApi`
- `@a2ui/lit/v0_9` — `A2uiController`, `Context`
- `@a2ui/web_core/types/types` — `Types` (네임스페이스, `MarkdownRenderer` 타입 포함)

### 저장소 내부 모듈
- [`../basic-catalog-a2ui-lit-element.ts`](../basic-catalog-a2ui-lit-element.ts.md) — `BasicCatalogA2uiLitElement`
- [`../../../directives/directives.ts`](../../../directives/directives.ts.md) — `markdown`

## Exports

- `A2uiBasicTextElement` — 클래스 (커스텀 엘리먼트)
- `A2uiText` — 상수 (객체: `TextApi` 스프레드 + `tagName: 'a2ui-basic-text'`)

## 상세 명세

### `A2uiBasicTextElement` 클래스

**데코레이터**: `@customElement('a2ui-basic-text')`

**상속**: `BasicCatalogA2uiLitElement<typeof TextApi>`

#### 정적 필드

- `static styles`: `CSSResult`:
  - `:host`: `display: inline-block`, `color: var(--_a2ui-text-color, var(--a2ui-text-color-text, var(--a2ui-color-on-background)))` — 부모(예: Button)가 `--_a2ui-text-color`를 설정하면 그것이 우선 적용된다.
  - `p, h1~h6, ol, ul, li, blockquote, pre`: `margin: var(--_a2ui-text-margin, 0)`.
  - `h1~h5`: `font-family: --a2ui-font-family-title`, `line-height: --a2ui-line-height-headings`(기본 `1.2`).
  - `h1`: `font-size: --a2ui-font-size-2xl`; `h2`: `--a2ui-font-size-xl`; `h3`: `--a2ui-font-size-l`; `p, h4`: `--a2ui-font-size-m`; `h5`: `--a2ui-font-size-s`.
  - `p, ol, ul, li, blockquote, .a2ui-caption`: `line-height: --a2ui-line-height-body`(기본 `1.5`).
  - `.a2ui-caption, .a2ui-caption > *, .a2ui-caption ::slotted(*)`: `font-size: --a2ui-font-size-xs`, `color: --a2ui-text-caption-color`(기본 `light-dark(#666, #aaa)`).
  - `a`: `color: --a2ui-text-a-color`(기본 `inherit`), `font-weight: --a2ui-text-a-font-weight`(기본 `inherit`).

#### 인스턴스 필드

- `@consume({context: Context.markdown, subscribe: true}) accessor markdownRenderer: Types.MarkdownRenderer | undefined` — 애플리케이션이 컨텍스트로 제공하는 마크다운 렌더러. `subscribe: true`로 컨텍스트 값이 변경될 때 자동으로 업데이트된다. 없으면 `undefined`.

#### `createController(): A2uiController`

`new A2uiController(this, TextApi)`를 반환.

#### `render(): TemplateResult | typeof nothing`

1. `this.controller.props`가 없으면 `nothing` 반환.
2. `props.text`를 문자열로 변환하여 `markdownText`에 저장 (`props.text`가 문자열이면 그대로, 아니면 `String(props.text ?? '')`).
3. `props.variant`에 따라 `markdownText` 앞에 마크다운 헤딩 접두어를 추가:
   - `'h1'` → `# ` 접두어, `'h2'` → `## `, `'h3'` → `### `, `'h4'` → `#### `, `'h5'` → `##### `.
   - `'body'`, `'caption'`, 그 외 → 변경 없음.
4. `markdown(markdownText, this.markdownRenderer)`를 호출하여 렌더링 결과 `renderedMarkdown` 획득.
5. `props.variant === 'caption'`이면 `<span class="a2ui-caption">{renderedMarkdown}</span>` 반환.
6. 그 외이면 `html\`${renderedMarkdown}\`` 반환.

### `A2uiText` 상수

`TextApi` 스프레드 + `tagName: 'a2ui-basic-text'`.

## 동작 흐름

컨텍스트에서 마크다운 렌더러 주입(없으면 undefined) → props에서 텍스트와 variant 수신 → variant에 따라 텍스트에 마크다운 헤딩 접두어 추가 → `markdown` 디렉티브로 렌더링 → `caption` variant이면 `.a2ui-caption` 클래스 span으로 래핑, 그 외이면 그대로 반환. 헤딩 스타일은 CSS로, caption 특수 스타일은 CSS 클래스로 처리하여 마크다운 파이프라인을 통일한다.
