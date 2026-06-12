# renderers/lit/src/0.8/ui/icon.ts

## 개요

`<a2ui-icon>` Web Component를 정의한다. Material Symbols Outlined 폰트를 이용해 아이콘을 렌더링하며, 아이콘 이름은 리터럴 문자열 또는 데이터 바인딩 경로(path) 방식의 `Primitives.StringValue`를 통해 지정할 수 있다. 이름에 포함된 대문자는 snake_case로 자동 변환된다.

## 의존성

### 외부 패키지
- `lit` — `html`, `css`, `nothing`
- `lit/decorators.js` — `customElement`, `property`
- `lit/directives/class-map.js` — `classMap`
- `lit/directives/style-map.js` — `styleMap`

### 저장소 내부 모듈
- [`./root.js`](../root.ts.md) — 베이스 클래스 `Root`
- [`./styles.js`](../styles.ts.md) — `structuralStyles`
- `@a2ui/web_core/data/model-processor` — `A2uiMessageProcessor` (외부 패키지)
- `@a2ui/web_core/types/primitives` — `Primitives` 네임스페이스 (외부 패키지)

## Exports

| 이름 | 종류 | 설명 |
|---|---|---|
| `Icon` | 클래스 (LitElement 서브클래스) | `<a2ui-icon>` 커스텀 엘리먼트 |

## 상세 명세

### `@customElement('a2ui-icon') class Icon extends Root`

**Properties (Lit @property)**

- `accessor name: Primitives.StringValue | null = null` — 아이콘 이름. `StringValue`는 `{ literalString: string }`, `{ literal: string }`, `{ path: string }` 중 하나일 수 있다.

**Static 필드**

- `static styles: CSSResultGroup` — `structuralStyles`와 컴포넌트 로컬 CSS 배열.
  - `:host`: `display: block`, `flex: var(--weight)`, `min-height: 0`.
  - `.g-icon`: Material Symbols Outlined 폰트 패밀리, 24px, `font-feature-settings: 'liga'` 포함 아이콘 렌더링에 필요한 전체 폰트 설정.

---

#### `#renderIcon(): TemplateResult | typeof nothing`

Private 메서드. 아이콘 이름을 해석해 `<span class="g-icon">` 형태로 반환한다.

내부 헬퍼 클로저 `render(url: string)`:
- `url`에 포함된 대문자를 `url.replace(/([A-Z])/gm, '_$1').toLocaleLowerCase()`로 snake_case로 변환한다.
- `html\`<span class="g-icon">${url}</span>\``를 반환한다.

처리 흐름:
1. `this.name`이 null이면 `nothing` 반환.
2. `this.name`이 객체이고 `'literalString' in this.name`이면 `this.name.literalString ?? ''`을 `render`에 전달.
3. 그렇지 않고 `'literal' in this.name`이면 `this.name.literal ?? ''`을 `render`에 전달.
4. 그렇지 않고 `'path' in this.name && this.name.path`이면:
   - `this.processor` 또는 `this.component`가 없으면 `html\`(no model)\`` 반환.
   - `this.processor.getData(this.component, this.name.path, surfaceId)`로 값을 가져온다. `surfaceId`는 `this.surfaceId ?? A2uiMessageProcessor.DEFAULT_SURFACE_ID`.
   - 값이 없거나 문자열이 아니면 `html\`Invalid icon name\`` 반환.
   - 문자열이면 `render(iconName)` 호출.
5. 위 조건을 모두 만족하지 않으면 `html\`(empty)\`` 반환.

---

#### `render(): TemplateResult`

1. `<section>` 엘리먼트를 반환한다.
2. `class`는 `classMap(this.theme.components.Icon)`.
3. `style`은 `this.theme.additionalStyles?.Icon`이 있으면 `styleMap(...)`, 없으면 `nothing`.
4. 내부에 `${this.#renderIcon()}`을 삽입한다.

## 동작 흐름

컴포넌트 초기화 시 `name` 속성을 받는다. `render` 호출 시 `#renderIcon`이 `name`의 타입을 분기 처리해 아이콘 이름 문자열을 추출하고, 대문자를 snake_case로 변환한 후 Material Symbols 폰트 span으로 감싸 반환한다. 데이터 바인딩 경로인 경우 `processor`를 통해 런타임에 값을 조회한다.
