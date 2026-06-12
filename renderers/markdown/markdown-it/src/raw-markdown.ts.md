# renderers/markdown/markdown-it/src/raw-markdown.ts

## 개요

이 파일은 markdown-it 라이브러리를 래핑한 `MarkdownItRenderer` 클래스와 그 싱글턴 인스턴스 `rawMarkdownRenderer`를 제공한다. `@a2ui/web_core`의 `MarkdownRendererTagClassMap`을 통해 렌더링 시 특정 HTML 태그에 CSS 클래스를 동적으로 주입하는 기능을 지원한다. 이 클래스는 HTML sanitization을 수행하지 않으며, sanitize 책임은 호출자(`markdown.ts`)가 진다.

## 의존성

### 외부 패키지

- `markdown-it` — 마크다운을 HTML로 변환하는 핵심 파서/렌더러 라이브러리.
- `@a2ui/web_core` — `Types.MarkdownRendererTagClassMap` 타입을 위해 임포트한다.

### 저장소 내부 모듈

없음.

## Exports

- `MarkdownItRenderer` (클래스) — markdown-it 렌더러 래퍼 클래스.
- `rawMarkdownRenderer` (상수, `MarkdownItRenderer` 인스턴스) — 모듈 로드 시 생성된 싱글턴 인스턴스.

## 상세 명세

### 클래스 `MarkdownItRenderer`

**필드:**
- `private markdownIt` — `markdownit()`으로 생성된 markdown-it 인스턴스. 기본 옵션(프리셋 없음)으로 초기화된다.

**생명주기:**
1. 생성자에서 `this.registerTagClassMapRules()`를 호출하여 커스텀 렌더러 규칙을 등록한다.

---

#### `private registerTagClassMapRules(): void`

**동작 로직:**

프록시할 규칙 이름 배열(`rulesToProxy`)을 정의한다. 포함되는 규칙은 다음 8가지다: `'paragraph_open'`, `'heading_open'`, `'bullet_list_open'`, `'ordered_list_open'`, `'list_item_open'`, `'link_open'`, `'strong_open'`, `'em_open'`.

각 `ruleName`에 대해 루프를 돌면서:
1. `this.markdownIt.renderer.rules[ruleName]`에 저장된 기존 규칙 함수를 `originalRule`에 캐시한다. 기본값이 없으면 `undefined`가 된다.
2. 같은 슬롯에 새 함수를 등록한다. 이 함수는 `(tokens, idx, options, env, self)` 시그니처를 가지며:
   - `tokens[idx]`에서 현재 토큰(`token`)을 가져온다.
   - `env?.tagClassMap`을 `Types.MarkdownRendererTagClassMap | undefined`로 읽는다.
   - `tagClassMap`이 존재하면 `token.tag`(예: `'h1'`, `'p'`)를 키로 클래스 배열을 조회하고, 배열의 각 `clazz`에 대해 `token.attrJoin('class', clazz)`를 호출해 클래스를 누적 병합한다. 해당 태그에 대한 항목이 없으면 빈 배열(`[]`)을 기본값으로 사용한다.
   - `originalRule`이 존재하면 `originalRule(tokens, idx, options, env, self)`을 호출하여 원래 렌더링 결과를 반환한다.
   - `originalRule`이 없으면 `self.renderToken(tokens, idx, options)`를 폴백으로 호출한다.

이 접근 방식은 markdown-it의 [렌더러 규칙 오버라이드 패턴](https://github.com/markdown-it/markdown-it/blob/master/docs/examples/renderer_rules.md#to-add-a-default-css-class-to-an-element)을 따른다.

---

#### `render(value: string, tagClassMap?: Types.MarkdownRendererTagClassMap): string`

**매개변수:**
- `value: string` — 렌더링할 마크다운 원문.
- `tagClassMap?: Types.MarkdownRendererTagClassMap` — 태그명을 키, 클래스명 배열을 값으로 하는 선택적 맵.

**반환 타입:** `string` — 렌더링된 HTML 문자열(sanitize되지 않음).

**동작 로직:**
`this.markdownIt.render(value, {tagClassMap})`을 호출한다. `tagClassMap`은 markdown-it의 `env` 객체로 전달되며, 위에서 등록한 프록시 규칙에서 `env?.tagClassMap`으로 접근된다. `tagClassMap`이 `undefined`이면 클래스 주입이 일어나지 않는다. 각 `render` 호출은 독립적이므로 이전 호출의 `tagClassMap`이 유지되지 않는다(무상태).

---

### 상수 `rawMarkdownRenderer`

`new MarkdownItRenderer()`로 생성된 모듈 레벨 싱글턴 인스턴스. 모듈 import 시 즉시 생성된다. `markdown.ts`에서 이 인스턴스를 직접 가져다 사용한다.

## 동작 흐름

1. 모듈 로드 시 `MarkdownItRenderer` 인스턴스가 생성되고, 생성자에서 8개의 "_open" 규칙이 프록시된다.
2. `render(value, tagClassMap)` 호출 시 markdown-it이 마크다운을 파싱하고 토큰 트리를 생성한다.
3. 각 "_open" 토큰 렌더링 시 프록시 함수가 실행되어 `tagClassMap` 기반으로 클래스를 주입한 뒤 원래 규칙(또는 기본 renderToken)으로 HTML을 생성한다.
4. 최종 HTML 문자열이 반환된다(sanitize 없음).
