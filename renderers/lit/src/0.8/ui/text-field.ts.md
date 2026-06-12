# renderers/lit/src/0.8/ui/text-field.ts

## 개요

`TextField`는 단일 행 텍스트 입력 커스텀 엘리먼트(`a2ui-textfield`)다. `Root`를 상속하며, `text` 프로퍼티의 리터럴 또는 데이터 바인딩 경로에서 초기 값을 추출하고, 사용자 입력 시 `A2uiMessageProcessor`를 통해 모델에 값을 기록한다. 선택적 레이블, 유효성 검사 정규식(`validationRegexp`), 숫자/텍스트 입력 타입 전환(`textFieldType`)을 지원하며, 입력 이벤트마다 `A2UIValidationEvent`를 발행하여 외부 리스너가 유효성 상태를 감지할 수 있도록 한다.

## 의존성

### 외부 패키지
- `lit` — `html`, `css`, `nothing`
- `lit/decorators.js` — `customElement`, `property`
- `lit/directives/class-map.js` — `classMap`
- `lit/directives/style-map.js` — `styleMap`
- `@a2ui/web_core/data/model-processor` — `A2uiMessageProcessor`
- `@a2ui/web_core/types/primitives` — `Primitives.StringValue`
- `@a2ui/web_core/types/types` — `Types.ResolvedTextField`
- `@a2ui/web_core` — `Events` 네임스페이스 (`Events.A2UIValidationEvent`)

### 저장소 내부 모듈
- [`./root.js`](./root.ts.md) — `Root` (기반 클래스)
- [`./styles.js`](./styles.ts.md) — `structuralStyles`
- [`./utils/utils.js`](./utils/utils.ts.md) — `extractStringValue`

## Exports

| 이름 | 종류 |
|---|---|
| `TextField` | 클래스 (`@customElement('a2ui-textfield')`, `Root` 상속) |

## 상세 명세

### 클래스: `TextField`

`Root`를 상속하며 `@customElement('a2ui-textfield')`로 등록된다.

#### 필드 및 프로퍼티

| 이름 | 타입 | 기본값 | 데코레이터 | 설명 |
|---|---|---|---|---|
| `text` | `Primitives.StringValue \| null` | `null` | 입력 필드의 현재 값. 리터럴 또는 바인딩 경로. `@property()` |
| `label` | `Primitives.StringValue \| null` | `null` | 입력 필드 위에 표시할 레이블. `@property()` |
| `textFieldType` | `Types.ResolvedTextField['textFieldType'] \| null` | `null` | 입력 타입. `'number'`이면 `type="number"`, 그 외는 `type="text"`. `@property()` |
| `validationRegexp` | `string \| null` | `null` | HTML `pattern` 어트리뷰트에 사용할 정규식 문자열. `@property()` |

#### `static styles`

`structuralStyles`와 인라인 `css` 블록을 배열로 결합한다.
- `* { box-sizing: border-box; }`
- `:host { display: flex; flex: var(--weight); }`
- `input { display: block; width: 100%; }`
- `input:invalid { border-color: var(--color-error); color: var(--color-error); outline-color: var(--color-error); }`
- `input:invalid:focus { border-color: var(--color-error); outline-color: var(--color-error); }`
- `label { display: block; margin-bottom: 4px; }`

---

#### `private #setBoundValue(value: string): void`

바인딩 경로를 통해 모델에 값을 저장하는 메서드. 다음 조건 중 하나라도 해당하면 즉시 반환(no-op):
1. `this.text`가 falsy이거나 `this.processor`가 없음
2. `this.text`에 `'path'` 키가 없음
3. `this.text.path`가 falsy

모든 조건을 통과하면 `this.processor.setData(this.component, this.text.path, value, this.surfaceId ?? A2uiMessageProcessor.DEFAULT_SURFACE_ID)`를 호출하여 데이터를 저장한다.

---

#### `private #renderField(value: string | number, label: string): TemplateResult`

입력 UI 구조를 렌더링한다.

1. `<section class=${classMap(this.theme.components.TextField.container)}>` 래퍼.
2. `label`이 빈 문자열이 아닌 경우: `<label class=${classMap(this.theme.components.TextField.label)} for="data">${label}</label>` 렌더링. 빈 문자열이면 `nothing`.
3. `<input>`:
   - `autocomplete="off"`, `id="data"`, `name="data"`
   - `.value=${value}`, `.placeholder=${'Please enter a value'}`
   - `pattern=${this.validationRegexp || nothing}` — `validationRegexp`가 있으면 HTML `pattern` 어트리뷰트 적용
   - `type=${this.textFieldType === 'number' ? 'number' : 'text'}`
   - `class=${classMap(this.theme.components.TextField.element)}`
   - `style`: `this.theme.additionalStyles?.TextField`가 있으면 `styleMap()`, 없으면 `nothing`
   - `@input` 핸들러:
     1. `evt.target`이 `HTMLInputElement`인지 확인. 아니면 반환.
     2. `new Events.A2UIValidationEvent({ componentId: this.id, value: evt.target.value, valid: evt.target.checkValidity() })`를 `this.dispatchEvent()`로 발행한다.
     3. `this.#setBoundValue(evt.target.value)`를 호출하여 모델에 값을 저장한다.

---

#### `render(): TemplateResult`

`extractStringValue(this.label, this.component, this.processor, this.surfaceId)`로 레이블 문자열을 추출하고, `extractStringValue(this.text, this.component, this.processor, this.surfaceId)`로 현재 값 문자열을 추출한 뒤, `this.#renderField(value, label)`을 호출하여 반환한다.

## 동작 흐름

`root.ts`가 `<a2ui-textfield>`를 생성하고 `label`, `text`, `textFieldType`, `validationRegexp`를 설정한다. `render()`가 `extractStringValue`로 레이블과 값을 해석(리터럴/경로 모두 처리)하여 `#renderField()`를 호출한다. `#renderField()`가 `<input>` 엘리먼트를 렌더링하며, `validationRegexp`가 있으면 브라우저 기본 유효성 검사가 동작하고 `input:invalid` CSS 규칙에 의해 에러 스타일이 표시된다. 사용자가 입력할 때마다 `A2UIValidationEvent`가 발행되어 외부 유효성 상태 추적이 가능하고, `#setBoundValue()`로 모델 데이터가 동시에 갱신된다.
