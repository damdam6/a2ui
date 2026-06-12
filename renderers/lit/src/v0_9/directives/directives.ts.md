# renderers/lit/src/v0_9/directives/directives.ts

## 개요

`directives` 디렉터리에서 제공하는 Lit 커스텀 디렉티브들을 집합적으로 re-export하는 배럴(barrel) 파일이다. 현재는 `markdown` 디렉티브 하나만 re-export한다.

## 의존성

### 저장소 내부 모듈
- [`./markdown.js`](./markdown.ts.md) — `markdown` 디렉티브 함수

## Exports

- `markdown` (함수/디렉티브): `MarkdownDirective`를 감싸는 Lit 디렉티브 팩토리 함수 (re-export)

## 동작 흐름

단순 re-export 파일로 자체 로직이 없다. 소비자는 `import { markdown } from '.../directives/directives.js'`로 마크다운 디렉티브를 임포트한다.
