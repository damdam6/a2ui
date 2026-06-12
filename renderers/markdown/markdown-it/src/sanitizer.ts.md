# renderers/markdown/markdown-it/src/sanitizer.ts

## 개요

이 파일은 DOMPurify를 사용하여 HTML 문자열에서 XSS 위험 요소를 제거하는 `sanitize` 함수를 제공한다. 브라우저(DOMPurify가 `window`를 직접 인식)와 Node.js(jsdom의 `window`를 `globalThis.window`에 주입) 양쪽 환경에서 동작하는 아이소모픽 구현이다. `isomorphic-dompurify` 패키지 대신 직접 초기화 로직을 구현하여 Angular 샘플과의 빌드 호환성 문제를 회피한다.

## 의존성

### 외부 패키지

- `dompurify` — HTML 문자열 sanitize에 사용하는 핵심 라이브러리.

### 저장소 내부 모듈

없음.

## Exports

- `sanitize` (함수) — HTML 문자열을 받아 sanitize된 HTML 문자열을 반환한다.

## 상세 명세

### 모듈 레벨 변수 `purify`

타입: `any`. 초기값 `undefined`. DOMPurify 인스턴스를 지연 초기화하여 캐시하는 변수다. 최초 `sanitize` 호출 시 환경에 따라 한 번만 초기화되며, 이후 호출에서는 이미 초기화된 인스턴스를 재사용한다.

---

### `sanitize(html: string): string`

**매개변수:**
- `html: string` — sanitize할 HTML 문자열.

**반환 타입:** `string` — DOMPurify에 의해 정제된 HTML 문자열.

**동작 로직(단계별):**

1. **지연 초기화 분기:** `purify`가 falsy(undefined)이면 초기화 블록에 진입한다.
   - **분기 A — 브라우저 환경:** `typeof DOMPurify.sanitize === 'function'`이 참이면 `purify = DOMPurify`로 설정한다. DOMPurify가 브라우저의 전역 `window`를 자동으로 사용하는 경우다.
   - **분기 B — Node.js 환경:** 분기 A가 거짓이면 `(globalThis as any).window`를 `globalWindow`로 읽는다.
     - `globalWindow`가 존재하면 `DOMPurify(globalWindow)`를 호출하여 jsdom 기반 인스턴스를 생성하고 `purify`에 저장한다.
     - `globalWindow`도 없으면 `'DOMPurify requires a window object. If testing, provide a jsdom window as `globalThis`.'` 메시지와 함께 `Error`를 throw한다.
2. **sanitize 실행:** `purify.sanitize(html)`을 호출하고 결과를 반환한다.

**경계 케이스:**
- Node.js 테스트 환경에서는 `markdown.test.ts`처럼 `globalThis.window`에 jsdom의 window를 주입해야 에러 없이 동작한다.
- `purify`는 모듈 생명주기 동안 한 번만 초기화되므로, 환경이 바뀌어도 재초기화되지 않는다.

## 동작 흐름

1. `sanitize(html)` 최초 호출 시 `purify` 초기화: 브라우저이면 DOMPurify 직접 사용, Node이면 `globalThis.window`로 DOMPurify 팩토리 호출, 둘 다 없으면 에러.
2. 이후 호출에서는 초기화 단계를 건너뛰고 `purify.sanitize(html)` 바로 실행.
3. sanitize된 문자열 반환.
