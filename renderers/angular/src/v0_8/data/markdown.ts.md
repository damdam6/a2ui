# renderers/angular/src/v0_8/data/markdown.ts

## 개요

마크다운 문자열을 HTML로 변환하는 Angular 서비스 계층을 정의한다. 추상 클래스 `MarkdownRenderer`가 계약을 정의하고, `DefaultMarkdownRenderer`가 선택적 패키지 `@a2ui/markdown-it`를 동적으로 로드하여 구현한다. `provideMarkdownRenderer` 팩토리 함수를 통해 외부 렌더러 함수나 기본 구현을 DI 토큰으로 등록할 수 있다.

## 의존성

### 외부 패키지
- `@angular/core` — `Injectable`
- `@a2ui/markdown-it` (선택적 피어 의존성, 동적 import) — `renderMarkdown`

### 저장소 내부 모듈
- [`../types`](../types.ts.md) — `MarkdownRenderer as MarkdownRendererType`, `MarkdownRendererOptions` (타입 전용)

## Exports

| 이름 | 종류 |
|---|---|
| `MarkdownRenderer` | 추상 클래스 (Angular injectable) |
| `DefaultMarkdownRenderer` | 구체 클래스 (Angular injectable) |
| `provideMarkdownRenderer` | 함수 |

## 상세 명세

### `MarkdownRenderer` 추상 클래스

`@Injectable({ providedIn: 'root' })` 데코레이터를 가진 추상 클래스. DI 토큰 역할을 겸한다.

#### `render(markdown: string, options?: MarkdownRendererOptions): Promise<string>` (abstract)

구현 클래스가 반드시 제공해야 하는 메서드. 마크다운 문자열을 받아 HTML 문자열을 비동기로 반환해야 한다.

---

### `DefaultMarkdownRenderer` 클래스

`MarkdownRenderer`를 상속하는 `@Injectable({ providedIn: 'root' })` 클래스.

#### `warningLogged` (private static)

경고 메시지가 이미 출력되었는지 추적하는 클래스 수준 플래그. 초기값 `false`. 한 번 경고가 출력되면 `true`로 변경되어 이후 호출에서 반복 경고를 방지한다.

#### `render(markdown: string, options?: MarkdownRendererOptions): Promise<string>` (override async)

1. `try` 블록에서 `await import('@a2ui/markdown-it')`로 선택적 패키지를 동적 로드하고 `renderMarkdown(markdown, options)` 결과를 반환.
2. `catch` 블록에서 `DefaultMarkdownRenderer.warningLogged`가 `false`이면 `console.warn`으로 경고 메시지를 출력하고 플래그를 `true`로 설정.
3. 폴백으로 원본 `markdown` 문자열을 그대로 반환 (v0.8 기본 구현).

---

### `provideMarkdownRenderer(renderFn?: MarkdownRendererType)`

**매개변수**
- `renderFn` (선택): `MarkdownRendererType` 타입의 외부 렌더러 함수.

**동작 로직**
- `renderFn`이 제공된 경우: `{provide: MarkdownRenderer, useValue: {render: renderFn}}` 객체를 반환하여 사용자 정의 렌더러를 DI에 등록.
- `renderFn`이 없는 경우: `{provide: MarkdownRenderer, useClass: DefaultMarkdownRenderer}`를 반환하여 기본 구현을 등록.

## 동작 흐름

앱 시작 시 `provideMarkdownRenderer`를 providers에 추가하면 `MarkdownRenderer` 토큰이 DI 트리에 등록된다. `Text` 컴포넌트 등이 `inject(MarkdownRenderer)`로 서비스를 주입받아 `render()` 메서드를 호출하면, 기본 구현은 `@a2ui/markdown-it` 패키지를 동적 로드하여 처리하고 패키지가 없으면 원본 마크다운 문자열을 반환한다.
