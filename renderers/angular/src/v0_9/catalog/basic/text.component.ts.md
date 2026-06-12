# renderers/angular/src/v0_9/catalog/basic/text.component.ts

## 개요

A2UI v0.9 Text 컴포넌트의 Angular 구현체다. 텍스트를 `variant` 값에 따라 heading(`h1`~`h5`), caption, 또는 Markdown 본문으로 렌더링한다. Markdown 렌더링은 `MarkdownRenderer` 서비스를 통해 비동기로 처리되며, 경쟁 조건(race condition) 방지를 위해 요청 ID 기반의 최신 응답 추적 로직을 내장한다.

## 의존성

### 외부 패키지
- `@angular/core`: `Component`, `computed`, `ChangeDetectionStrategy`, `inject`, `signal`, `effect`
- `@a2ui/web_core/v0_9/basic_catalog`: `TextApi`

### 저장소 내부 모듈
- [`../../core/markdown`](../../core/markdown.ts.md): `MarkdownRenderer` 추상 클래스 (DI)
- [`./basic-catalog-component`](./basic-catalog-component.ts.md): 기반 추상 클래스

## Exports

- `TextComponent` — Angular standalone 컴포넌트 클래스

## 상세 명세

### `TextComponent`

**데코레이터**: `@Component`
- `selector`: `'a2ui-v09-text'`
- `standalone: true`
- `changeDetection`: `ChangeDetectionStrategy.OnPush`
- 스타일은 `:host ::ng-deep`을 사용(아래 설명)

**상속**: `BasicCatalogComponent<typeof TextApi>`

**정적 필드**

#### `NON_MARKDOWN_VARIANTS: Set<string>` (private static readonly)
값: `new Set(['h1', 'h2', 'h3', 'h4', 'h5', 'caption'])`. 이 set에 포함된 variant는 Markdown 렌더링을 수행하지 않는다.

**인스턴스 필드 및 computed 속성**

| 이름 | 종류 | 타입 | 설명 |
|------|------|------|------|
| `markdownRenderer` | private inject | `MarkdownRenderer` | 마크다운 렌더링 서비스 |
| `variant` | computed Signal | `string` | `props()['variant']?.value() \|\| 'body'`. 기본값 `'body'` |
| `text` | computed Signal | `string` | `props()['text']?.value() \|\| ''`. 기본값 빈 문자열 |
| `isNonMarkdownVariant` | computed Signal | `boolean` | `NON_MARKDOWN_VARIANTS.has(variant())` |
| `resolvedText` | WritableSignal | `string` | 비동기 마크다운 렌더링 결과. 초기값 `''` |
| `renderRequestId` | private | `number` | 진행 중인 렌더링 요청의 순번. 초기값 `0` |

**생성자 (`constructor()`)**

`super()`를 호출한 뒤 `effect()`를 등록한다. 효과(effect) 로직:
1. `isNonMarkdownVariant()`가 `true`이면 즉시 반환(마크다운 렌더링 불필요).
2. `text()`를 읽고 `++renderRequestId`로 새 요청 ID를 생성한다.
3. `markdownRenderer.render(text)`를 호출하여 Promise를 시작한다.
4. Promise가 resolve되면, 현재 `renderRequestId`가 요청 시점의 ID와 같을 때만 `resolvedText.set(rendered)`를 호출한다. 이는 빠르게 연속으로 text가 변경될 때 오래된 응답이 최신 응답을 덮어쓰지 않도록 보호한다.

**템플릿 구조**

- `isNonMarkdownVariant()`가 `true`이면 `<span class="a2ui-text {variant}">`를 렌더링하고, `@switch(variant())`로 `h1`~`h5` 또는 `em`(caption) 엘리먼트를 내부에 렌더링한다.
- 그 외(Markdown 경로)이면 `<span class="a2ui-text {variant}" [innerHTML]="resolvedText()">`를 렌더링한다. `innerHTML`을 사용하므로 Angular ViewEncapsulation이 주입된 HTML에 적용되지 않는다.

**스타일 (`:host ::ng-deep` 사용 이유 및 CSS 변수)**

`innerHTML`로 주입된 HTML 엘리먼트에는 Angular가 컴파일 타임에 생성하는 범위 속성이 없어서 일반 뷰 캡슐화가 적용되지 않는다. `:host ::ng-deep`을 사용하여 스타일을 컴포넌트 범위 내에서 주입된 HTML까지 적용한다.

| CSS 변수 | 기본값 | 역할 |
|----------|--------|------|
| `--a2ui-text-color-text` | `var(--a2ui-color-on-background)` | 기본 텍스트 색상 |
| `--a2ui-font-family-title` | `inherit` | 제목 폰트 패밀리 |
| `--a2ui-line-height-headings` | `1.2` | 제목 줄 높이 |
| `--a2ui-line-height-body` | `1.5` | 본문 줄 높이 |
| `--a2ui-text-caption-color` | `light-dark(#666, #aaa)` | caption 색상 |
| `--a2ui-text-a-color` | `inherit` | 링크 색상 |
| `--a2ui-text-a-font-weight` | `inherit` | 링크 폰트 굵기 |
| `--a2ui-font-size-2xl` | — | h1 폰트 크기 |
| `--a2ui-font-size-xl` | — | h2 폰트 크기 |
| `--a2ui-font-size-l` | — | h3 폰트 크기 |
| `--a2ui-font-size-m` | — | h4, p 폰트 크기 |
| `--a2ui-font-size-s` | — | h5 폰트 크기 |
| `--a2ui-font-size-xs` | — | caption 폰트 크기 |

## 동작 흐름

컴포넌트 생성 시 `effect()`가 등록되어 `text` 또는 `isNonMarkdownVariant`가 변경될 때마다 실행된다. variant가 heading 또는 caption이면 즉시 `text()`를 직접 렌더링한다. Markdown 경로이면 비동기 렌더링이 시작되고, 결과가 도착하면 `resolvedText` signal이 갱신되어 Angular signal 변경 감지를 트리거한다. 경쟁 조건 보호로 인해 마지막으로 시작된 렌더링만 실제로 적용된다.
