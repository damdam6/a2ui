# renderers/lit/src/0.8/ui/text.ts

## 개요

`Text`는 텍스트 렌더링 커스텀 엘리먼트(`a2ui-text`)다. `Root`를 상속하며, `StringValue` 형태의 텍스트(리터럴 또는 데이터 바인딩 경로)를 해석하고 `usageHint`(`h1`~`h5`, `caption`, `body`)에 따라 Markdown 접두어를 붙여 커스텀 `markdown` 디렉티브로 렌더링한다. 추가 스타일은 `HintedStyles` 구조를 통해 `usageHint`별로 다른 값을 가질 수 있다.

## 의존성

### 외부 패키지
- `lit` — `html`, `css`, `nothing`
- `lit/decorators.js` — `customElement`, `property`
- `@lit/context` — `consume`
- `lit/directives/class-map.js` — `classMap`
- `lit/directives/style-map.js` — `styleMap`
- `@a2ui/web_core/data/model-processor` — `A2uiMessageProcessor`
- `@a2ui/web_core/types/primitives` — `Primitives.StringValue`
- `@a2ui/web_core/types/types` — `Types.ResolvedText`, `Types.MarkdownRenderer`

### 저장소 내부 모듈
- [`./root.js`](./root.ts.md) — `Root` (기반 클래스)
- [`./styles.js`](./styles.ts.md) — `structuralStyles`
- [`./directives/directives.js`](./directives/directives.ts.md) — `markdown` 디렉티브
- [`./context/context.js`](./context/context.ts.md) — `Context.markdown` (컨텍스트 키)
- [`../index.js`](../index.ts.md) — `Styles` 네임스페이스 (`Styles.merge`, `Styles.appendToAll`)

## Exports

| 이름 | 종류 |
|---|---|
| `Text` | 클래스 (`@customElement('a2ui-text')`, `Root` 상속) |

## 상세 명세

### 인터페이스: `HintedStyles` (비공개)

파일 내부에서만 사용하는 인터페이스.

필드: `h1`, `h2`, `h3`, `h4`, `h5`, `body`, `caption` — 각각 `Record<string, string>` 타입의 인라인 스타일 맵.

`theme.additionalStyles.Text`가 이 형태인지 판별하는 `#areHintedStyles()` 타입 가드의 반환 타입으로 사용된다. 참고: `#areHintedStyles`의 검사 목록에는 `'h6'`도 포함되어 있으나 인터페이스 멤버에는 `h6`가 없다.

---

### 클래스: `Text`

`Root`를 상속하며 `@customElement('a2ui-text')`로 등록된다.

#### 필드 및 프로퍼티

| 이름 | 타입 | 기본값 | 데코레이터 | 설명 |
|---|---|---|---|---|
| `text` | `Primitives.StringValue \| null` | `null` | 렌더링할 텍스트. `@property()` |
| `usageHint` | `Types.ResolvedText['usageHint'] \| null` | `null` | 시맨틱 힌트. 허용 값: `'h1'`~`'h5'`, `'caption'`, `'body'`, null. `@property({reflect: true, attribute: 'usage-hint'})` |
| `markdownRenderer` | `Types.MarkdownRenderer \| undefined` | `undefined` | `@consume({context: Context.markdown})`로 주입되는 외부 Markdown 렌더러. |

#### `static styles`

`structuralStyles`와 인라인 `css` 블록을 배열로 결합한다.
- `:host { display: block; flex: var(--weight); }`
- `h1, h2, h3, h4, h5 { line-height: inherit; font: inherit; }` — 브라우저 기본 헤딩 스타일을 초기화

---

#### `private #renderText(): TemplateResult | string`

텍스트 값을 추출하고 Markdown 형식으로 변환하여 반환하는 핵심 메서드.

**텍스트 추출** (`textValue` 초기값 `null`):
- `this.text`가 객체인 경우:
  1. `'literalString' in this.text && this.text.literalString` → 해당 문자열.
  2. `'literal' in this.text && this.text.literal !== undefined` → 해당 값.
  3. `'path' in this.text && this.text.path` → `this.processor.getData(this.component, this.text.path, this.surfaceId ?? A2uiMessageProcessor.DEFAULT_SURFACE_ID)` 조회.
     - `this.processor` 또는 `this.component`가 없으면 `` html`(no model)` `` 즉시 반환.
     - 결과가 non-null/undefined이면 `.toString()` 변환.
- `textValue`가 `null` 또는 `undefined`이면 `` html`(empty)` `` 반환.

**Markdown 접두어 추가** (`markdownText = textValue`):

| `usageHint` 값 | 변환 결과 |
|---|---|
| `'h1'` | `# ${markdownText}` |
| `'h2'` | `## ${markdownText}` |
| `'h3'` | `### ${markdownText}` |
| `'h4'` | `#### ${markdownText}` |
| `'h5'` | `##### ${markdownText}` |
| `'caption'` | `*${markdownText}*` (이탤릭) |
| 그 외 / null | 변환 없음 (body) |

최종 반환: `markdown(markdownText, this.markdownRenderer, { tagClassMap: Styles.appendToAll(this.theme.markdown, ['ol', 'ul', 'li'], {}) })`를 Lit 템플릿으로 감싸 반환. 테마의 마크다운 클래스 맵에 `ol`, `ul`, `li` 태그 클래스를 병합하여 전달한다.

---

#### `private #areHintedStyles(styles: unknown): styles is HintedStyles`

타입 가드. 아래 세 조건을 모두 만족해야 `true`:
1. `typeof styles === 'object'`
2. `Array.isArray(styles)`가 `false`
3. `styles`가 `null`이 아님
4. `['h1', 'h2', 'h3', 'h4', 'h5', 'h6', 'caption', 'body']` 키를 모두 포함

---

#### `private #getAdditionalStyles(): Record<string, string>`

`theme.additionalStyles?.Text`를 기반으로 인라인 스타일 맵을 반환한다.
1. `styles`가 없으면 빈 객체 `{}` 반환.
2. `#areHintedStyles(styles)`가 `true`이면: `this.usageHint ?? 'body'`로 힌트 키를 결정하고 `styles[hint]`를 `Record<string, string>`으로 반환.
3. `false`이면: `styles`를 `Record<string, string>`으로 그대로 반환.

---

#### `render(): TemplateResult`

1. `Styles.merge(this.theme.components.Text.all, this.usageHint ? this.theme.components.Text[this.usageHint] : {})`로 CSS 클래스 맵을 병합한다.
2. `<section class=${classMap(classes)} style=${...}>` 내부에 `#renderText()` 결과를 배치한다.
3. `style`: `this.theme.additionalStyles?.Text`가 있으면 `styleMap(this.#getAdditionalStyles())`, 없으면 `nothing`.

## 동작 흐름

`root.ts`가 `<a2ui-text>`를 생성하고 `text`와 `usageHint`를 설정한다. `render()`가 `usageHint`를 기반으로 테마 CSS 클래스와 추가 스타일을 결정한다. `#renderText()`가 `text`에서 실제 문자열을 추출하고 `usageHint`에 따라 Markdown 접두어를 붙인다. `markdown` 디렉티브가 Markdown 문자열을 HTML로 렌더링하며 테마의 목록 요소 클래스가 적용된다. 데이터 바인딩 경로인 경우, signal 기반으로 `processor`의 데이터가 변경되면 `render()`가 재실행되어 텍스트가 최신 상태로 갱신된다.
