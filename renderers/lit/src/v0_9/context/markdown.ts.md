# renderers/lit/src/v0_9/context/markdown.ts

## 개요

Lit의 Context API를 이용해 마크다운 렌더러를 컴포넌트 트리에 주입하기 위한 컨텍스트 토큰을 정의하는 파일이다. `Text` 위젯이 마크다운 콘텐츠를 렌더링할 때 이 컨텍스트를 통해 실제 렌더러 함수를 획득한다. 컨텍스트 키는 `Symbol('A2UIMarkdown')`으로 고유하게 식별된다.

## 의존성

### 외부 패키지
- `@lit/context`: `createContext`
- `@a2ui/web_core/types/types`: `Types` 네임스페이스 (타입 전용)

## Exports

- `markdown` (상수): `Context<Types.MarkdownRenderer | undefined>` 타입의 컨텍스트 토큰

## 상세 명세

### `markdown` 상수

- 타입: `Context<Types.MarkdownRenderer | undefined>`
- `createContext<Types.MarkdownRenderer | undefined>(Symbol('A2UIMarkdown'))`로 생성한다.
- 컨텍스트 키로 전역 고유 `Symbol('A2UIMarkdown')`을 사용하여 이름 충돌을 방지한다.
- 값 타입을 `undefined`도 허용하므로, 마크다운 렌더러가 제공되지 않은 상태를 정상 케이스로 처리할 수 있다.

## 동작 흐름

공급자 컴포넌트가 `@provide({ context: markdown })` 데코레이터를 통해 `Types.MarkdownRenderer` 함수(비동기 마크다운-to-HTML 변환 함수)를 트리에 주입한다. 소비자 컴포넌트(주로 `Text` 위젯)는 `@consume({ context: markdown })` 또는 `MarkdownDirective` 내부에서 이 컨텍스트 토큰으로 렌더러를 조회하여 실제 마크다운 변환에 활용한다.
