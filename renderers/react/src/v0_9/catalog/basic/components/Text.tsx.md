# renderers/react/src/v0_9/catalog/basic/components/Text.tsx

## 개요

`Text` 컴포넌트는 a2ui basic catalog의 `TextApi` 스키마를 따르는 React 구현체다. `variant` 값에 따라 두 가지 렌더링 경로로 분기된다: `h1`~`h5`와 `caption`은 네이티브 HTML 시맨틱 요소로, 나머지는 Markdown 파이프라인(`useMarkdown`)을 통해 HTML로 변환하여 렌더링한다. CSS Module(`Text.module.css`)로 스타일 클래스를 적용한다.

## 의존성

### 외부 패키지
- `react` — `React.CSSProperties`
- `@a2ui/web_core/v0_9/basic_catalog` — `TextApi`
- CSS Module: `./Text.module.css` — `styles.a2uiText`, `styles.a2uiCaption`

### 저장소 내부 모듈
- [`../../../adapter`](../../../adapter.tsx.md) — `createComponentImplementation`
- [`../utils`](../utils.ts.md) — `getBaseLeafStyle`, `getWeightStyle`, `useBasicCatalogStyles`
- [`../hooks/useMarkdown`](../hooks/useMarkdown.ts.md) — `useMarkdown`

## Exports

| 이름 | 종류 |
|------|------|
| `Text` | 상수 (ReactComponentImplementation) |

## 상세 명세

### `NON_MARKDOWN_VARIANTS` (모듈 레벨 상수)

`const NON_MARKDOWN_VARIANTS = new Set(['h1', 'h2', 'h3', 'h4', 'h5', 'caption'])`

Markdown 파이프라인을 거치지 않고 네이티브 HTML 요소로 직접 렌더링할 variant 집합이다.

### `MarkdownText` (내부 함수형 컴포넌트)

**시그니처:** `function MarkdownText({ text, className, style }: { text: string; className: string; style: React.CSSProperties }): JSX.Element`

**동작:**
1. `useMarkdown(text)`를 호출하여 `renderedHtml: string | null`을 받는다.
2. `renderedHtml`이 `null`이면(렌더러 미설정 상태) className에 `' no-markdown-renderer'`를 추가한다.
3. `renderedHtml`이 `null`이 아니면 `dangerouslySetInnerHTML={{ __html: renderedHtml }}`을 contentProps로, `null`이면 `children: text`를 contentProps로 사용한다.
4. `<div className={classes} style={style} {...contentProps} />`를 반환한다.

### `NonMarkdownText` (내부 함수형 컴포넌트)

**시그니처:** `function NonMarkdownText({ text, variant, className, style }: { text: string; variant: string; className: string; style: React.CSSProperties }): JSX.Element`

**동작:**
1. `variant === 'caption'`이면 `isCaption = true`, `HeadingTag = 'em'`으로 설정한다.
2. 그 외의 경우 `HeadingTag`를 `variant`(즉, `'h1'`~`'h5'`)로 설정한다.
3. `isCaption`이면 `<span className style><em>{text}</em></span>` 반환.
4. 그 외는 `<div className style><h1..h5>{text}</h1..h5></div>` 반환.

### `Text`

`createComponentImplementation(TextApi, renderFn)` 호출로 생성된 컴포넌트 구현체다.

**렌더 함수 시그니처:** `({props}) => JSX.Element`

**매개변수 (props 필드):**
- `props.text` — 표시할 텍스트. 문자열이 아닌 경우 `String(props.text ?? '')`로 변환
- `props.variant` — 텍스트 변형 (`'h1'`~`'h5'`, `'caption'`, `'body'` 등, 옵셔널)
- `props.weight` — flex weight 숫자 (옵셔널). `getWeightStyle`에 전달됨

**동작 로직:**
1. `useBasicCatalogStyles()`로 global 스타일 주입.
2. `text` 값을 정규화한다: `typeof props.text === 'string'`이면 그대로, 아니면 `String(props.text ?? '')`.
3. `style`을 구성한다: `getBaseLeafStyle()`(`{boxSizing: 'border-box'}`)에 `getWeightStyle(props.weight)` 결과를 병합.
4. `variant`가 존재하고 `NON_MARKDOWN_VARIANTS`에 포함되면:
   - `isCaption = variant === 'caption'`
   - `className = [styles.a2uiText, isCaption ? styles.a2uiCaption : variant].join(' ')`
   - `<NonMarkdownText text={text} variant={variant} className={className} style={style} />` 반환
5. 그 외:
   - `className = [styles.a2uiText, variant || 'body'].join(' ')`
   - `<MarkdownText text={text} className={className} style={style} />` 반환

## 동작 흐름

props 업데이트 시, variant가 `NON_MARKDOWN_VARIANTS` 집합에 속하면 `NonMarkdownText`가 동기적으로 렌더링된다. 그 외에는 `MarkdownText`가 `useMarkdown` hook을 통해 비동기적으로 HTML을 렌더링하며, 렌더러가 없을 경우 plain text로 fallback하고 CSS 클래스에 `no-markdown-renderer`가 추가된다.
