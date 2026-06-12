# renderers/angular/src/v0_9/core/markdown.ts

## 개요

A2UI Angular 렌더러에서 마크다운 텍스트를 HTML로 변환하는 추상 레이어를 정의한다. 선택적 피어 디펜던시인 `@a2ui/markdown-it` 패키지를 동적으로 로드하며, 로드에 실패할 경우 원본 마크다운을 그대로 반환하는 폴백 동작을 제공한다. Angular DI를 통해 커스텀 렌더러를 주입할 수 있도록 팩토리 함수도 제공한다.

## 의존성

### 외부 패키지
- `@angular/core` — `Injectable`
- `@a2ui/markdown-it` — 동적 `import()`로 로드되는 선택적 피어 디펜던시 (`renderMarkdown` 함수 사용)

### 저장소 내부 모듈

없음.

## Exports

- `MarkdownRendererOptions` (인터페이스)
- `MarkdownRenderer` (추상 클래스)
- `DefaultMarkdownRenderer` (클래스)
- `provideMarkdownRenderer` (함수)

## 상세 명세

### `interface MarkdownRendererOptions`

마크다운 렌더링 옵션을 나타내는 인터페이스.

| 필드 | 타입 | 설명 |
|------|------|------|
| `tagClassMap` | `Record<string, string> \| undefined` | HTML 태그 이름을 CSS 클래스명에 매핑하는 선택적 딕셔너리 |

---

### `abstract class MarkdownRenderer`

마크다운 렌더러의 추상 베이스 클래스. Angular DI 토큰 역할을 겸한다.

**추상 메서드 `render(markdown: string, options?: MarkdownRendererOptions): Promise<string>`**
- 마크다운 문자열을 받아 HTML 문자열을 비동기로 반환한다.
- 하위 클래스가 반드시 구현해야 한다.

---

### `class DefaultMarkdownRenderer extends MarkdownRenderer`

`@Injectable({ providedIn: 'root' })` 데코레이터로 전역 싱글턴으로 제공된다.

**정적 필드**
- `private static warningLogged: boolean` — 폴백 경고 메시지가 이미 콘솔에 출력되었는지 추적하는 플래그. 경고가 반복 출력되지 않도록 방지한다.

**메서드 `override async render(markdown: string, options?: MarkdownRendererOptions): Promise<string>`**

동작 단계:
1. `import('@a2ui/markdown-it')`를 동적으로 시도하여 `renderMarkdown` 함수를 가져온다.
2. 성공 시 `renderMarkdown(markdown, options as any)`를 호출하고 그 결과를 반환한다.
3. 로드 실패 시 `catch` 블록 진입. `warningLogged`가 `false`이면 `console.warn`으로 경고 메시지 `'[DefaultMarkdownRenderer] Failed to load optional `@a2ui/markdown-it` renderer. Using fallback.'`을 출력하고 `warningLogged`를 `true`로 설정한다.
4. 폴백: 원본 `markdown` 문자열을 그대로 반환한다.

---

### `function provideMarkdownRenderer(renderFn?: (markdown: string, options?: MarkdownRendererOptions) => Promise<string>)`

**반환 타입**: Angular Provider 객체 (`{ provide, useValue }` 또는 `{ provide, useClass }`)

동작 단계:
1. `renderFn`이 제공된 경우: `{ provide: MarkdownRenderer, useValue: { render: renderFn } }` 형태의 provider 객체를 반환하여 임의의 함수를 렌더러로 등록한다.
2. `renderFn`이 `undefined`인 경우: `{ provide: MarkdownRenderer, useClass: DefaultMarkdownRenderer }`를 반환하여 기본 구현체를 사용하도록 한다.

## 동작 흐름

`MarkdownRenderer`는 DI 토큰이자 추상 클래스 역할을 한다. `DefaultMarkdownRenderer`가 기본 구현체이며, `@a2ui/markdown-it` 패키지를 런타임에 동적으로 불러오는 방식으로 선택적 의존성을 처리한다. 애플리케이션 모듈에서 `provideMarkdownRenderer()`를 호출하여 커스텀 렌더러를 주입하거나 기본 구현체를 사용할 수 있다.
