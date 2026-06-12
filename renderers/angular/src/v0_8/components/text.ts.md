# renderers/angular/src/v0_8/components/text.ts

## 개요

텍스트 콘텐츠를 마크다운으로 렌더링하는 Angular standalone 컴포넌트다. `usageHint`(h1~h5, body, caption)에 따라 원본 문자열을 마크다운 문법으로 변환한 뒤 `MarkdownRenderer`에 위임하고, 비동기 결과를 `AsyncPipe`를 통해 `innerHTML`로 주입한다. 테마 클래스 및 추가 스타일도 `usageHint`에 맞게 동적으로 적용한다.

## 의존성

### 외부 패키지
- `@angular/core` — `ChangeDetectionStrategy`, `Component`, `computed`, `inject`, `input`, `ViewEncapsulation`
- `@angular/common` — `AsyncPipe`
- `@a2ui/web_core/types/primitives` — `StringValue` 타입 (as `Primitives`)
- `@a2ui/web_core/styles/index` — `appendToAll`, `merge` 유틸리티 (as `Styles`)

### 저장소 내부 모듈
- [`../rendering/dynamic-component`](../rendering/dynamic-component.ts.md) — 기반 클래스 `DynamicComponent`
- [`../types`](../types.ts.md) — 타입 `ResolvedText`, `TextNode`
- [`../data/markdown`](../data/markdown.ts.md) — `MarkdownRenderer`

## Exports

| 이름 | 종류 |
|---|---|
| `Text` | Angular 컴포넌트 클래스 |

## 상세 명세

### `HintedStyles` 인터페이스 (파일 내부, non-export)

`h1`, `h2`, `h3`, `h4`, `h5`, `body`, `caption` 키 각각에 `Record<string, string>` 타입의 스타일 맵을 갖는 구조체. `additionalStyles`가 hint별 스타일 맵인지 판별하는 데 사용된다.

---

### `Text` 클래스

`DynamicComponent<TextNode>`를 상속하는 Angular 컴포넌트.

**데코레이터 메타데이터**
- `selector`: `'a2ui-text'`
- `changeDetection`: `ChangeDetectionStrategy.OnPush`
- `encapsulation`: `ViewEncapsulation.None` (전역 CSS 스코프)
- `template`: `<section [class]="classes()" [style]="additionalStyles()" [innerHTML]="resolvedText() | async">`
- `styles`: `a2ui-text`를 `display: block; flex: var(--weight)`로, 내부 h1~h5 요소는 `line-height: inherit; font: inherit`으로 설정.
- `imports`: `[AsyncPipe]`

---

#### 입력 신호(Inputs)

| 이름 | 타입 | 기본값 | 필수 |
|---|---|---|---|
| `text` | `Primitives.StringValue \| null` | — | 필수(`input.required`) |
| `usageHint` | `ResolvedText['usageHint'] \| null` | `null` | 선택 |

---

#### `markdownRenderer` (private)

`inject(MarkdownRenderer)`로 주입되는 서비스 인스턴스.

---

#### `resolvedText` (protected computed)

`usageHint`와 `text` 신호를 읽어 `Promise<string>`을 반환하는 계산 신호.

1. `super.resolvePrimitive(this.text())`로 원시 문자열 `value`를 구한다.
2. `value`가 `null` 또는 `undefined`이면 `Promise.resolve('')` 반환.
3. `usageHint` 값에 따라 마크다운 문법 접두사/감싸기를 적용:
   - `'h1'` → `# value`
   - `'h2'` → `## value`
   - `'h3'` → `### value`
   - `'h4'` → `#### value`
   - `'h5'` → `##### value`
   - `'caption'` → `*value*`
   - 기타/null → `String(value)` (변환 없음)
4. `this.markdownRenderer.render(value, { tagClassMap: Styles.appendToAll(this.theme.markdown, ['ol', 'ul', 'li'], {}) })`를 호출하고 그 `Promise`를 반환.

---

#### `classes` (protected computed)

`usageHint`를 읽어 `Styles.merge(this.theme.components.Text.all, usageHint ? this.theme.components.Text[usageHint] : {})` 결과를 반환한다. 기본 클래스와 hint별 클래스를 병합하는 역할.

---

#### `additionalStyles` (protected computed)

`this.theme.additionalStyles?.Text` 값이 없으면 `null` 반환. 있으면:
1. `this.areHintedStyles(styles)`가 `true`이면 `(styles as any)[usageHint ?? 'body'] || {}`를 사용.
2. 그렇지 않고 일반 객체이면 `styles as Record<string, string>`을 그대로 사용.
3. 최종 `additionalStyles` 객체를 반환.

---

#### `areHintedStyles(styles: unknown): styles is HintedStyles` (private)

타입 가드. `styles`가 비null 비배열 객체이고, `['h1', 'h2', 'h3', 'h4', 'h5', 'h6', 'caption', 'body']` 모든 키를 포함하는지 `every`로 검사하여 `boolean` 반환.

## 동작 흐름

컴포넌트가 렌더링되면 `resolvedText`가 `Promise<string>`을 생성하고, `AsyncPipe`가 이를 구독하여 HTML 문자열로 변환해 `innerHTML`에 주입한다. `usageHint`가 바뀌면 세 개의 계산 신호(클래스, 스타일, 텍스트)가 모두 재평가되며, OnPush 변경 감지로 불필요한 재렌더링을 방지한다.
