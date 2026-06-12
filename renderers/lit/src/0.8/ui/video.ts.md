# renderers/lit/src/0.8/ui/video.ts

## 개요

`a2ui-video` 커스텀 엘리먼트를 정의하는 Lit 컴포넌트 파일이다. `Root` 기반 클래스를 상속하며, `url` 프로퍼티로 받은 `Primitives.StringValue`를 해석해 HTML `<video controls>` 태그를 렌더링한다. 리터럴 문자열(`literalString`), 리터럴 객체(`literal`), 데이터 경로 참조(`path`) 세 가지 URL 표현 방식을 모두 처리하며, 오류 상황에는 인라인 메시지를 표시한다.

## 의존성

### 외부 패키지

- `lit` — `html`, `css`, `nothing`
- `lit/decorators.js` — `customElement`, `property`
- `lit/directives/class-map.js` — `classMap`
- `lit/directives/style-map.js` — `styleMap`
- `@a2ui/web_core/data/model-processor` — `A2uiMessageProcessor`
- `@a2ui/web_core/types/primitives` — `Primitives`

### 저장소 내부 모듈

- [`./root.js`](./root.ts.md) — `Root` (부모 클래스)
- [`./styles.js`](./styles.ts.md) — `structuralStyles`

## Exports

| 이름 | 종류 |
|---|---|
| `Video` | 클래스 (커스텀 엘리먼트 `a2ui-video`) |

## 상세 명세

### 클래스: `Video`

- **선언:** `@customElement('a2ui-video') export class Video extends Root`

#### 필드

- `url: Primitives.StringValue | null` — `@property()` 데코레이터 적용, 기본값 `null`. 비디오 소스 URL. 세 가지 형식(`literalString` 객체, `literal` 객체, `path` 객체) 중 하나 또는 `null`

#### 스타일 (`static styles`)

`structuralStyles`와 컴포넌트 전용 CSS의 배열이다. 컴포넌트 전용 CSS:
- 모든 요소에 `box-sizing: border-box`
- `:host`: `display: block`, `flex: var(--weight)`, `min-height: 0`, `overflow: auto`
- `video`: `display: block`, `width: 100%`

#### 비공개 메서드: `#renderVideo()`

- **시그니처:** `#renderVideo(): TemplateResult | typeof nothing`
- **동작:**
  1. `this.url`이 falsy이면 `nothing`을 반환한다.
  2. `this.url`이 객체 타입인 경우 세 가지 필드를 순서대로 확인한다:
     - `literalString` 키가 있으면: `<video controls src=${this.url.literalString} />`를 반환한다.
     - `literal` 키가 있으면: `<video controls src=${this.url.literal} />`를 반환한다.
     - `path` 키가 있고 truthy이면:
       - `this.processor` 또는 `this.component`가 없으면 `html\`(no processor)\``를 반환한다.
       - `this.processor.getData(this.component, this.url.path, this.surfaceId ?? A2uiMessageProcessor.DEFAULT_SURFACE_ID)`로 URL을 조회한다.
       - 조회 결과가 없거나(falsy) `string`이 아니면 `html\`Invalid video URL\``을 반환한다.
       - 유효한 문자열이면 `<video controls src=${videoUrl} />`를 반환한다.
  3. 어떤 분기에도 해당하지 않으면 `html\`(empty)\``를 반환한다.

#### 공개 메서드: `render()`

- **시그니처:** `render(): TemplateResult`
- **동작:** `<section>` 태그 안에 `#renderVideo()`의 결과를 삽입한다.
  - `<section>`의 `class`는 `classMap(this.theme.components.Video)`로 테마 클래스를 적용한다.
  - `<section>`의 `style`은 `this.theme.additionalStyles?.Video`가 존재하면 `styleMap(...)`으로 적용하고, 없으면 `nothing`이다.

## 동작 흐름

컴포넌트가 연결된 후 `url` 프로퍼티가 설정되면 Lit이 `render()`를 호출한다. `render()`는 테마 클래스가 적용된 `<section>` 래퍼 안에 `#renderVideo()`의 결과를 배치한다. URL 해석은 `literalString` → `literal` → `path` 순서로 진행되며, `path` 방식에서만 `A2uiMessageProcessor`를 통한 런타임 조회가 발생한다. 오류 케이스는 모두 사용자에게 인라인 텍스트 메시지로 표시된다.
