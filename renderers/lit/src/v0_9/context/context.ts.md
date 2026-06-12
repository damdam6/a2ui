# renderers/lit/src/v0_9/context/context.ts

## 개요

Lit 렌더러에서 의존성 주입에 사용하는 컨텍스트 객체를 모아 re-export하는 네임스페이스 파일이다. 현재는 마크다운 렌더러 컨텍스트(`markdown`)만 포함하며, 이를 `Context` 객체의 프로퍼티로 묶어 단일 진입점을 제공한다.

## 의존성

### 저장소 내부 모듈
- [`./markdown.js`](./markdown.ts.md) — `markdown` 컨텍스트 토큰

## Exports

- `Context` (상수 객체): `{ markdown }` 형태의 네임스페이스 객체

## 상세 명세

### `Context` 상수 객체

- 타입: `{ markdown: Context<Types.MarkdownRenderer | undefined> }`
- `markdown` 프로퍼티에 `./markdown.js`에서 가져온 `markdown` 컨텍스트 토큰을 그대로 할당한다.
- 소비자가 `Context.markdown`으로 마크다운 렌더러를 주입하거나 조회할 수 있다.

## 동작 흐름

파일은 얇은 집합 레이어로, 직접적인 로직 없이 `markdown` 컨텍스트를 `Context` 네임스페이스로 노출한다. 사용 측에서는 `import { Context } from '@a2ui/lit/v0_9'` 후 `Context.markdown`을 Lit의 `@provide` / `@consume` 데코레이터에 전달하여 마크다운 렌더러를 공급 또는 소비한다.
