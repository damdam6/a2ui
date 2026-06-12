# renderers/lit/src/0.8/ui/tabs.ts

## 개요

`Tabs`는 탭 네비게이션 커스텀 엘리먼트(`a2ui-tabs`)다. `Root`를 상속하며, `titles` 배열로 탭 버튼을 생성하고 `selected` 인덱스로 활성 탭을 추적한다. `willUpdate`에서 Light DOM 자식의 `slot` 어트리뷰트를 직접 조작하여 활성 탭 컨텐츠만 `name="current"` 슬롯에 투영되도록 한다. 탭 타이틀은 리터럴 문자열 또는 데이터 바인딩 경로(`path`)를 통해 동적으로 해석된다.

## 의존성

### 외부 패키지
- `lit` — `html`, `css`, `PropertyValues`, `nothing`
- `lit/decorators.js` — `customElement`, `property`
- `lit/directives/repeat.js` — `repeat`
- `lit/directives/class-map.js` — `classMap`
- `lit/directives/style-map.js` — `styleMap`
- `@a2ui/web_core/data/model-processor` — `A2uiMessageProcessor`
- `@a2ui/web_core/types/primitives` — `Primitives.StringValue`

### 저장소 내부 모듈
- [`./root.js`](./root.ts.md) — `Root` (기반 클래스)
- [`./styles.js`](./styles.ts.md) — `structuralStyles`
- [`../index.js`](../index.ts.md) — `Styles` 네임스페이스 (`Styles.merge` 사용)

## Exports

| 이름 | 종류 |
|---|---|
| `Tabs` | 클래스 (`@customElement('a2ui-tabs')`, `Root` 상속) |

## 상세 명세

### 클래스: `Tabs`

`Root`를 상속하며 `@customElement('a2ui-tabs')`로 등록된다.

#### 필드 및 프로퍼티

| 이름 | 타입 | 기본값 | 데코레이터 | 설명 |
|---|---|---|---|---|
| `titles` | `Primitives.StringValue[] \| null` | `null` | 탭 버튼에 표시할 타이틀 배열. `@property()` |
| `selected` | `number` | `0` | 현재 선택된 탭의 인덱스. `@property()` |

#### `static styles`

`structuralStyles`와 인라인 `css` 블록을 배열로 결합한다.
- `:host { display: block; flex: var(--weight); }`

---

#### `protected willUpdate(changedProperties: PropertyValues<this>): void`

1. `super.willUpdate(changedProperties)`를 먼저 호출하여 부모(`Root`)의 Light DOM 렌더링을 실행한다.
2. `selected` 프로퍼티가 변경된 경우에만 아래를 실행한다:
   - `this.children`의 모든 자식 엘리먼트에서 `slot` 어트리뷰트를 `removeAttribute('slot')`으로 제거한다.
   - `this.children[this.selected]`로 새로 선택된 자식을 가져온다. 해당 인덱스에 자식이 없으면 즉시 반환한다.
   - 해당 자식의 `slot` 속성을 `'current'`로 설정하여 `<slot name="current">`에 투영되도록 한다.

---

#### `private #renderTabs(): TemplateResult | typeof nothing`

`this.titles`가 없으면 `nothing`을 반환한다.

`repeat()` 디렉티브로 `titles` 배열을 순회하며 각 탭 버튼을 렌더링한다.

**타이틀 문자열 해석** (순서대로 시도):
1. `'literalString' in title && title.literalString` → 해당 문자열 사용.
2. `'literal' in title && title.literal !== undefined` → 해당 값 사용.
3. `'path' in title && title.path` → `processor.getData(component, title.path, this.surfaceId ?? A2uiMessageProcessor.DEFAULT_SURFACE_ID)` 호출.
   - `this.processor` 또는 `this.component`가 없으면 `` html`(no model)` `` 즉시 반환.
   - 조회 결과가 `string`이 아니면 `` html`(invalid)` `` 반환.

**CSS 클래스 결정**:
- `selected === idx`(현재 활성 탭): `Styles.merge(this.theme.components.Tabs.controls.all, this.theme.components.Tabs.controls.selected)` 결과 사용.
- 비활성 탭: `{...this.theme.components.Tabs.controls.all}` (얕은 복사).

**`<button>` 렌더링**:
- `?disabled=${selected === idx}` — 현재 탭 버튼은 비활성화(disabled).
- `class=${classMap(classes)}`
- `@click` 핸들러: `this.selected = idx`로 설정.

반환: `<div id="buttons" class=${classMap(this.theme.components.Tabs.element)}>` 내부에 버튼 목록.

---

#### `private #renderSlot(): TemplateResult`

`` html`<slot name="current"></slot>` ``을 반환한다. 현재 선택된 탭의 자식 컨텐츠만 이 슬롯으로 투영된다.

---

#### `render(): TemplateResult`

`<section class=${classMap(this.theme.components.Tabs.container)} style=${...}>` 내부에 `[this.#renderTabs(), this.#renderSlot()]` 배열을 렌더링한다. `this.theme.additionalStyles?.Tabs`가 있으면 `styleMap()`, 없으면 `nothing`.

## 동작 흐름

`root.ts`가 `tabItems`에서 `titles` 배열과 `childComponents` 배열을 분리해 `<a2ui-tabs>`에 전달한다. `Root.willUpdate`가 `childComponents`를 Light DOM에 삽입하여 탭 컨텐츠 엘리먼트들이 자식 노드로 추가된다. `Tabs.willUpdate`가 `selected` 변경을 감지하여 해당 자식의 `slot="current"` 어트리뷰트를 설정하고 나머지 자식들의 슬롯을 제거한다. `#renderTabs()`가 탭 버튼 목록을 렌더링하고, `#renderSlot()`의 `<slot name="current">`가 활성 탭 컨텐츠를 투영한다. 사용자가 탭 버튼을 클릭하면 `selected`가 갱신되어 `willUpdate`가 재실행되고 표시 탭이 전환된다.
