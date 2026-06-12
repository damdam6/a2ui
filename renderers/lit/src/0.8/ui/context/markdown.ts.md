# renderers/lit/src/0.8/ui/context/markdown.ts

## 개요

`markdown.ts`는 A2UI의 마크다운 렌더러를 Lit 컨텍스트로 제공하기 위한 컨텍스트 심볼 정의 파일이다. `@lit/context`의 `createContext`를 사용해 `Types.MarkdownRenderer | undefined` 타입의 컨텍스트를 생성한다. 컴포넌트 트리 내에서 마크다운 렌더러 구현체를 의존성 주입 방식으로 공유하는 데 사용된다.

## 의존성

### 외부 패키지
- `@lit/context` — `createContext`
- `@a2ui/web_core/types/types` — `Types` (네임스페이스)

### 저장소 내부 모듈
없음.

## Exports

- `markdown` (상수, `Context<Types.MarkdownRenderer | undefined>`)

## 상세 명세

### 상수 `markdown`

- 타입: `Context<Types.MarkdownRenderer | undefined>`
- 값: `createContext<Types.MarkdownRenderer | undefined>(Symbol('A2UIMarkdown'))`
- 고유 식별자로 `Symbol('A2UIMarkdown')`을 사용한다. Symbol은 매번 새로운 고유 값을 생성하므로 이 모듈이 한 번만 로드되는 것이 중요하다.
- 소비하는 컴포넌트는 `@consume({context: markdown})`으로 마크다운 렌더러를 주입받는다.

## 동작 흐름

이 파일은 모듈 로드 시 단 한 번 `createContext(Symbol('A2UIMarkdown'))`를 실행해 컨텍스트 객체를 생성한다. 제공자(provider) 컴포넌트가 이 컨텍스트에 `MarkdownRenderer` 구현체를 제공하면, 하위 트리의 소비자(consumer) 컴포넌트들이 `@consume` 또는 `consume()` API를 통해 동일한 인스턴스를 사용할 수 있다.
