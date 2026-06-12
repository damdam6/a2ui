# renderers/lit/src/0.8/ui/directives/directives.ts

## 개요

`directives/directives.ts`는 `directives` 디렉터리의 진입점 모듈로, `context/markdown.ts`에서 정의된 `markdown` Lit 컨텍스트 객체를 재내보낸다. 소비자가 `./directives/directives.js` 경로를 통해 마크다운 컨텍스트에 접근할 수 있도록 경로를 단일화하는 역할을 한다.

## 의존성

### 외부 패키지
없음.

### 저장소 내부 모듈
- [`../context/markdown.js`](../context/markdown.ts.md) — `markdown` 컨텍스트 객체

## Exports

- `markdown` (named re-export, `../context/markdown.js`에서)

## 상세 명세

### 내용

`export {markdown} from './markdown.js'` 단 한 줄의 재내보내기 선언으로만 구성된다. 실제 import 경로는 `../context/markdown.js`가 아닌 같은 디렉터리의 `./markdown.js`를 가리키고 있으나, 이는 `directives` 디렉터리 내에 별도의 `markdown.ts` 파일이 존재함을 의미한다. 해당 파일이 `context/markdown.ts`의 컨텍스트를 재내보내거나 자체적으로 마크다운 관련 기능을 정의할 수 있다.

## 동작 흐름

이 파일은 로직을 전혀 포함하지 않는다. 임포트 시 `./markdown.js`가 로드되어 `markdown` 심볼이 이 모듈의 네임스페이스로 노출된다.
