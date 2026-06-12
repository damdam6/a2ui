# renderers/markdown/markdown-it/src/markdown.ts

## 개요

이 파일은 마크다운 문자열을 안전한 HTML 문자열로 변환하는 비동기 공개 API인 `renderMarkdown` 함수를 제공한다. 내부적으로 `raw-markdown.ts`의 `rawMarkdownRenderer`로 HTML을 생성한 뒤, `sanitizer.ts`의 `sanitize`로 XSS를 방어한다. 렌더링과 위생 처리를 한 함수로 합성하는 얇은 파사드 계층이다.

## 의존성

### 저장소 내부 모듈

- [`./raw-markdown.js`](./raw-markdown.ts.md) — `rawMarkdownRenderer` 인스턴스를 가져온다.
- [`./sanitizer.js`](./sanitizer.ts.md) — `sanitize` 함수를 가져온다.

### 외부 패키지

- `@a2ui/web_core` — `Types.MarkdownRendererOptions` 타입을 위해 임포트한다.

## Exports

- `renderMarkdown` (비동기 함수) — 마크다운 문자열을 HTML로 변환하고 sanitize한 결과를 `Promise<string>`으로 반환한다.

## 상세 명세

### `renderMarkdown(value: string, options?: Types.MarkdownRendererOptions): Promise<string>`

**매개변수:**
- `value: string` — 렌더링할 마크다운 원문.
- `options?: Types.MarkdownRendererOptions` — 선택적 렌더러 옵션. 현재는 `tagClassMap` 필드를 사용한다.

**반환 타입:** `Promise<string>` — sanitize된 HTML 문자열로 resolve되는 프로미스.

**동작 로직:**
1. `rawMarkdownRenderer.render(value, options?.tagClassMap)`을 호출하여 HTML 문자열(`htmlString`)을 생성한다. `options`가 없거나 `tagClassMap`이 없으면 `undefined`가 전달된다.
2. `sanitize(htmlString)`을 호출하여 DOMPurify로 XSS 위험 요소를 제거한다.
3. sanitize된 결과 문자열을 `Promise`로 반환한다. 함수 자체는 `async`로 선언되어 있어 결과가 자동으로 프로미스로 래핑된다.

**경계 케이스:** `sanitize`가 `window` 객체 없이 호출되면 내부에서 에러를 던진다(sanitizer 명세 참조). `rawMarkdownRenderer.render`는 동기 함수이므로 실제 비동기 작업은 없다.

## 동작 흐름

```
마크다운 입력(value, options)
  → rawMarkdownRenderer.render(value, options?.tagClassMap)   [동기, HTML 생성]
  → sanitize(htmlString)                                      [동기, XSS 제거]
  → Promise<string> 반환
```
