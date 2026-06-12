# renderers/lit/src/0.8/ui/checkbox.ts

## 개요

`Checkbox` 클래스는 `a2ui-checkbox` 커스텀 엘리먼트를 구현하는 Lit 웹 컴포넌트다. A2UI 컴포넌트 모델의 `CheckBox` 노드를 렌더링하며, 불리언 값의 세 가지 형태(리터럴, 직접 리터럴, 데이터 경로 바인딩)를 지원한다. 체크박스 상태가 변경되면 `processor`를 통해 모델 데이터를 업데이트한다.

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
- [`./utils/utils.js`](./utils/utils.ts.md) — `extractStringValue`

## Exports

- `Checkbox` (클래스, `@customElement('a2ui-checkbox')` 데코레이터 적용)

## 상세 명세

### 클래스 `Checkbox extends Root`

`Root`를 상속하는 Lit 커스텀 엘리먼트. `@customElement('a2ui-checkbox')`로 등록된다.

#### 필드

| 필드 | 타입 | 기본값 | 설명 |
|------|------|--------|------|
| `value` | `Primitives.BooleanValue \| null` | `null` | `@property()` 데코레이터 적용. 체크박스 상태 값 객체. |
| `label` | `Primitives.StringValue \| null` | `null` | `@property()` 데코레이터 적용. 체크박스 레이블 값 객체. |

#### 정적 필드 `styles`

`structuralStyles`와 인라인 CSS를 배열로 조합한다. 인라인 CSS 규칙:
- `*` — `box-sizing: border-box`
- `:host` — `display: block`, `flex: var(--weight)`, `min-height: 0`, `overflow: auto`
- `input` — `display: block`, `width: 100%`
- `.description` — `font-size: 14px`, `margin-bottom: 4px`

#### 비공개 메서드 `#setBoundValue(value: boolean): void`

데이터 경로 바인딩된 `value`를 모델에 쓰는 내부 헬퍼.

1. `this.value`가 없거나 `this.processor`가 없으면 즉시 반환한다.
2. `this.value`에 `'path'` 키가 없으면 반환한다.
3. `this.value.path`가 falsy이면 반환한다.
4. `this.processor.setData(this.component, this.value.path, value, surfaceId)`를 호출해 값을 기록한다. `surfaceId`는 `this.surfaceId ?? A2uiMessageProcessor.DEFAULT_SURFACE_ID`를 사용한다.

#### 비공개 메서드 `#renderField(value: boolean | number): TemplateResult`

실제 체크박스 UI를 구성하는 내부 렌더링 헬퍼.

- `<section>`에 `classMap(this.theme.components.CheckBox.container)`와 조건부 `styleMap(this.theme.additionalStyles?.CheckBox)`를 적용한다.
- 내부에 `<input type="checkbox">`를 렌더링한다:
  - `class`는 `classMap(this.theme.components.CheckBox.element)` 적용.
  - `autocomplete="off"` 설정.
  - `@input` 핸들러: `evt.target`이 `HTMLInputElement`이면 `#setBoundValue(evt.target.checked)`를 호출한다.
  - `id="data"`와 `.checked=${value}`(Lit DOM 프로퍼티 바인딩)를 설정한다.
- `<label for="data">`에 `classMap(this.theme.components.CheckBox.label)`과 `extractStringValue(this.label, this.component, this.processor, this.surfaceId)`로 추출한 레이블 문자열을 삽입한다.

#### 메서드 `render(): TemplateResult | typeof nothing`

`this.value` 값의 형태에 따라 분기한다.

1. `this.value`가 객체이고 `'literalBoolean'` 키를 갖고 truthy이면 `#renderField(this.value.literalBoolean)`을 호출한다.
2. `'literal'` 키를 갖고 `undefined`가 아니면 `#renderField(this.value.literal)`을 호출한다.
3. `'path'` 키를 갖고 truthy이면:
   - `processor` 또는 `component`가 없으면 `(no model)` 텍스트를 반환한다.
   - `processor.getData(...)`로 불리언 값을 조회한다. 결과가 null이거나 `boolean`이 아니면 `Invalid label` 텍스트를 반환한다.
   - 정상 불리언이면 `#renderField(textValue)`를 호출한다.
4. 위 조건에 해당하지 않으면 `nothing`을 반환한다.

## 동작 흐름

`value`와 `label` 프로퍼티가 외부에서 바인딩되면 `render()`가 `value`의 형태를 판별해 `#renderField`를 호출한다. 사용자가 체크박스를 변경하면 `@input` 핸들러가 `#setBoundValue`를 호출해 모델을 업데이트하고, 모델 변경은 `processor`를 통해 상위 시스템으로 전파된다.
