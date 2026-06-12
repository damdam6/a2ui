# renderers/lit/src/0.8/ui/audio.ts

## 개요

`Audio` 클래스는 `a2ui-audioplayer` 커스텀 엘리먼트를 구현하는 Lit 웹 컴포넌트다. A2UI 컴포넌트 모델의 `AudioPlayer` 노드를 렌더링하며, URL 값의 세 가지 형태(리터럴 문자열, 직접 리터럴, 데이터 경로 바인딩)를 지원한다. 내부적으로 `<audio controls>` HTML 요소를 생성해 브라우저 기본 오디오 재생 컨트롤을 제공한다.

## 의존성

### 외부 패키지
- `lit` — `html`, `css`, `nothing`
- `lit/decorators.js` — `customElement`, `property`
- `lit/directives/class-map.js` — `classMap`
- `lit/directives/style-map.js` — `styleMap`
- `@a2ui/web_core/data/model-processor` — `A2uiMessageProcessor`
- `@a2ui/web_core/types/primitives` — `Primitives` (네임스페이스)

### 저장소 내부 모듈
- [`./root.js`](./root.ts.md) — 베이스 클래스 `Root`
- [`./styles.js`](./styles.ts.md) — `structuralStyles`

## Exports

- `Audio` (클래스, `@customElement('a2ui-audioplayer')` 데코레이터 적용)

## 상세 명세

### 클래스 `Audio extends Root`

`Root`를 상속하는 Lit 커스텀 엘리먼트. `@customElement('a2ui-audioplayer')`로 등록된다.

#### 필드

| 필드 | 타입 | 기본값 | 설명 |
|------|------|--------|------|
| `url` | `Primitives.StringValue \| null` | `null` | `@property()` 데코레이터 적용. 오디오 소스 URL을 나타내는 값 객체. |

#### 정적 필드 `styles`

`structuralStyles`와 인라인 CSS를 배열로 조합한다. 인라인 CSS 규칙:
- `*` — `box-sizing: border-box`
- `:host` — `display: block`, `flex: var(--weight)`, `min-height: 0`, `overflow: auto`
- `audio` — `display: block`, `width: 100%`

#### 비공개 메서드 `#renderAudio(): TemplateResult | typeof nothing`

`this.url` 값의 형태에 따라 분기 처리하여 `<audio controls>` 요소를 반환하는 내부 렌더링 헬퍼.

1. `this.url`이 `null`이면 `nothing`을 반환한다.
2. `this.url`이 객체이고 `'literalString'` 키를 갖는 경우, `this.url.literalString`을 `src`로 지정한 `<audio controls>` 태그를 반환한다.
3. `'literal'` 키를 갖는 경우, `this.url.literal`을 `src`로 지정한 `<audio controls>` 태그를 반환한다.
4. `'path'` 키를 갖고 값이 truthy인 경우:
   - `this.processor` 또는 `this.component`가 없으면 `(no processor)` 텍스트를 반환한다.
   - `this.processor.getData(this.component, this.url.path, surfaceId)`를 호출해 URL 문자열을 조회한다. `surfaceId`는 `this.surfaceId ?? A2uiMessageProcessor.DEFAULT_SURFACE_ID`를 사용한다.
   - 조회 결과가 없거나 문자열이 아니면 `Invalid audio URL` 텍스트를 반환한다.
   - 정상 문자열이면 해당 값을 `src`로 하는 `<audio controls>` 태그를 반환한다.
5. 위 조건에 해당하지 않으면 `(empty)` 텍스트를 반환한다.

#### 메서드 `render(): TemplateResult`

최상위 `<section>` 요소를 반환한다.
- `class`에는 `classMap(this.theme.components.AudioPlayer)`를 적용한다.
- `this.theme.additionalStyles?.AudioPlayer`가 존재하면 `styleMap`으로 인라인 스타일을 적용하고, 없으면 `nothing`을 사용한다.
- 내부에 `#renderAudio()`의 결과를 삽입한다.

## 동작 흐름

`url` 프로퍼티가 외부(보통 `Root.renderComponentTree`)에서 바인딩되면 Lit의 리액티브 시스템이 `render()`를 트리거한다. `render()`는 테마 클래스와 추가 스타일이 적용된 `<section>` 래퍼를 생성하고, 내부에서 `#renderAudio()`를 호출해 `url` 타입에 따른 적절한 `<audio>` 엘리먼트를 삽입한다. 데이터 경로 바인딩의 경우 `processor`를 통해 런타임 데이터를 조회한다.
