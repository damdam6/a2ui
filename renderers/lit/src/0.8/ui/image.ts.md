# renderers/lit/src/0.8/ui/image.ts

## 개요

`<a2ui-image>` Web Component를 정의한다. URL과 대체 텍스트를 `Primitives.StringValue` 형태(리터럴 또는 데이터 바인딩 경로)로 받아 `<img>` 태그를 렌더링한다. `usageHint`에 따라 테마의 용도별 클래스를 적용하고, `fit` 속성으로 CSS `object-fit` 값을 CSS 변수로 전달한다.

## 의존성

### 외부 패키지
- `lit` — `html`, `css`, `nothing`
- `lit/decorators.js` — `customElement`, `property`
- `lit/directives/class-map.js` — `classMap`
- `lit/directives/style-map.js` — `styleMap`

### 저장소 내부 모듈
- [`./root.js`](../root.ts.md) — 베이스 클래스 `Root`
- [`./styles.js`](../styles.ts.md) — `structuralStyles`
- [`../index.js`](../../index.ts.md) — `Styles` 네임스페이스 (`Styles.merge` 사용)
- `@a2ui/web_core/data/model-processor` — `A2uiMessageProcessor` (외부 패키지)
- `@a2ui/web_core/types/primitives` — `Primitives` 네임스페이스 (외부 패키지)
- `@a2ui/web_core/types/types` — `Types` 네임스페이스 (외부 패키지)

## Exports

| 이름 | 종류 | 설명 |
|---|---|---|
| `Image` | 클래스 (LitElement 서브클래스) | `<a2ui-image>` 커스텀 엘리먼트 |

## 상세 명세

### `@customElement('a2ui-image') class Image extends Root`

**Properties (Lit @property)**

- `accessor url: Primitives.StringValue | null = null` — 이미지 URL. `StringValue`는 `{ literalString }`, `{ literal }`, `{ path }` 중 하나.
- `accessor altText: Primitives.StringValue | null = null` — 대체 텍스트. 같은 `StringValue` 유니온 타입.
- `accessor usageHint: Types.ResolvedImage['usageHint'] | null = null` — 이미지 사용 용도 힌트. 테마에서 해당 키로 추가 클래스를 조회하는 데 사용된다.
- `accessor fit: 'contain' | 'cover' | 'fill' | 'none' | 'scale-down' | null = null` — CSS `object-fit` 값. null이면 기본값 `'fill'`.

**Static 필드**

- `static styles` — `structuralStyles`와 로컬 CSS 배열.
  - `:host`: `display: block`, `flex: var(--weight)`, `min-height: 0`, `overflow: auto`.
  - `img`: `display: block`, `width: 100%`, `height: 100%`, `object-fit: var(--object-fit, fill)`.

---

#### `#renderImage(): TemplateResult | typeof nothing`

Private 메서드. `url`을 해석해 `<img>` 태그를 반환한다.

내부 헬퍼 클로저 `render(url: string)`:
1. `resolvedAlt`를 빈 문자열로 초기화한다.
2. `this.altText`가 있고 객체이면:
   - `'literalString' in this.altText`이면 `altText.literalString ?? ''`.
   - `'literal' in this.altText`이면 `altText.literal ?? ''`.
   - `'path' in this.altText && altText.path`이면 `processor.getData(...)`로 값을 조회. 결과가 문자열이면 `resolvedAlt`에 대입.
3. `html\`<img src=${url} alt=${resolvedAlt} />\``를 반환한다.

URL 해석 흐름:
1. `this.url`이 null이면 `nothing` 반환.
2. `'literalString' in this.url`이면 `render(this.url.literalString ?? '')`.
3. `'literal' in this.url`이면 `render(this.url.literal ?? '')`.
4. `'path' in this.url && this.url.path`이면 `processor`/`component` 없으면 `html\`(no model)\`` 반환. 있으면 `processor.getData`로 URL 조회. 값이 없거나 문자열이 아니면 `html\`Invalid image URL\`` 반환. 문자열이면 `render(imageUrl)`.
5. 위 조건 불만족 시 `html\`(empty)\`` 반환.

---

#### `render(): TemplateResult`

1. `Styles.merge(this.theme.components.Image.all, this.usageHint ? this.theme.components.Image[this.usageHint] : {})`로 클래스 맵을 합산한다. `usageHint`가 있으면 해당 용도 클래스도 포함된다.
2. `<section>`을 반환하며:
   - `class`는 `classMap(classes)`.
   - `style`은 `styleMap({ ...(this.theme.additionalStyles?.Image ?? {}), '--object-fit': this.fit ?? 'fill' })`. `fit`이 null이면 CSS 변수 `--object-fit`은 `'fill'`이 된다.
3. 내부에 `${this.#renderImage()}`를 삽입한다.

## 동작 흐름

`url`, `altText`, `usageHint`, `fit` 속성이 변경되면 Lit이 `render`를 재호출한다. `render`는 `usageHint`를 고려한 클래스 맵을 계산하고, `--object-fit` CSS 변수를 설정한 후 `#renderImage`로 실제 `<img>` 태그를 생성한다. URL과 altText 모두 리터럴·바인딩 경로 두 가지 방식을 지원한다.
