# renderers/lit/src/0.8/ui/directives/markdown.ts

## 개요

마크다운 문자열을 HTML로 렌더링하는 Lit 커스텀 디렉티브를 정의한다. 주입된 `MarkdownRenderer` 함수가 있으면 그것을 사용하고, 없으면 선택적 피어 의존성 `@a2ui/markdown-it` 패키지의 동적 임포트를 시도하며, 둘 다 실패하면 원본 텍스트를 `<span class="no-markdown-renderer">`로 감싸는 폴백을 제공한다. `update` 훅에서 값과 옵션이 동일하면 `noChange`를 반환해 불필요한 재렌더링을 방지한다.

## 의존성

### 외부 패키지
- `lit` — `html`, `noChange`
- `lit/directive.js` — `Directive`, `DirectiveParameters`, `Part`, `directive`
- `lit/directives/unsafe-html.js` — `unsafeHTML`
- `lit/directives/until.js` — `until`
- `@a2ui/web_core/types/types` — `Types` 네임스페이스 (선택적 런타임 피어 `@a2ui/markdown-it`)

### 저장소 내부 모듈
없음 (web_core는 외부 패키지로 취급)

## Exports

| 이름 | 종류 | 설명 |
|---|---|---|
| `markdown` | 상수 (Lit directive 팩토리) | `MarkdownDirective`를 감싼 Lit `directive()` 결과물 |

## 상세 명세

### `class MarkdownDirective extends Directive`

Lit `Directive`의 서브클래스. 두 개의 private 필드로 렌더링 메모이제이션을 구현한다.

**Private 필드**

- `#lastValue: string | null = null` — 마지막으로 렌더링한 마크다운 문자열.
- `#lastTagClassMap: string | null = null` — 마지막으로 렌더링한 `markdownOptions.tagClassMap`의 JSON 직렬화 결과.

**Static 필드**

- `private static defaultMarkdownWarningLogged: boolean = false` — `@a2ui/markdown-it` 로드 실패 경고를 최초 1회만 출력하기 위한 플래그.

---

#### `update(_part: Part, [value, markdownRenderer, markdownOptions]: DirectiveParameters<this>): typeof noChange | TemplateResult`

Lit이 디렉티브를 재평가할 때 호출하는 메서드.

1. `markdownOptions?.tagClassMap`을 `JSON.stringify`해 `jsonTagClassMap`을 계산한다.
2. `#lastValue === value`이고 `jsonTagClassMap === #lastTagClassMap`이면 `noChange`를 반환해 DOM 업데이트를 건너뛴다.
3. 그렇지 않으면 `#lastValue`와 `#lastTagClassMap`을 갱신하고 `this.render(value, markdownRenderer, markdownOptions)`를 호출해 그 결과를 반환한다.

---

#### `render(value: string, markdownRenderer?: Types.MarkdownRenderer, markdownOptions?: Types.MarkdownRendererOptions): TemplateResult`

실제 렌더링 로직을 담당한다.

**경로 A — `markdownRenderer`가 주어진 경우:**
1. `markdownRenderer(value, markdownOptions)`를 호출해 Promise를 얻는다.
2. Promise가 resolve되면 반환된 HTML 문자열을 `unsafeHTML(value)`로 변환한다. (새니타이징은 렌더러 책임)
3. `until(rendered, html\`<span class="no-markdown-renderer">${value}</span>\`)` 형태로 반환해 렌더링 완료 전까지 원본 텍스트를 플레이스홀더로 표시한다.

**경로 B — `markdownRenderer`가 없는 경우 (동적 임포트):**
1. 즉시 실행 async 함수 `dynamicRendererPromise`를 생성한다.
2. 내부에서 `@a2ui/markdown-it` 패키지를 동적으로 `import`해 `renderMarkdown` 함수를 가져온다.
3. `renderMarkdown(value, markdownOptions)` 호출 결과를 `unsafeHTML`로 감싸 반환한다.
4. `import`나 `renderMarkdown` 호출이 실패하면 catch 블록에서 `defaultMarkdownWarningLogged`가 `false`일 때만 `console.warn`으로 경고를 출력하고 플래그를 `true`로 설정한 뒤, `html\`<span class="no-markdown-renderer">${value}</span>\``를 반환한다.
5. `until(dynamicRendererPromise, html\`<span class="no-markdown-renderer">${value}</span>\`)` 형태로 반환한다.

---

### `export const markdown`

`directive(MarkdownDirective)`의 반환값. Lit 템플릿에서 `markdown(value, renderer?, options?)` 형태로 호출한다.

## 동작 흐름

외부에서 `markdown(value, renderer, options)` 형태로 호출 → `update` 메서드가 값 변경 여부를 확인 → 변경 없으면 `noChange` 반환, 변경 있으면 `render` 실행 → renderer 주입 여부에 따라 Promise 경로 분기 → `until` 디렉티브가 플레이스홀더를 먼저 렌더하고 Promise resolve 시 최종 HTML로 교체.
