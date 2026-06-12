# renderers/lit/src/0.8/ui/slider.ts

## 개요

`Slider`는 범위 입력(`<input type="range">`) 커스텀 엘리먼트(`a2ui-slider`)다. `Root`를 상속하며, `value`에 리터럴 숫자(`literalNumber`/`literal`) 또는 데이터 바인딩 경로(`path`)를 받아 현재 값을 표시하고, 사용자가 슬라이더를 움직일 때 `A2uiMessageProcessor`를 통해 바인딩된 경로에 값을 기록한다. 선택적으로 레이블을 표시하며, 현재 슬라이더 수치를 `<span>`으로 함께 렌더링한다.

## 의존성

### 외부 패키지
- `lit` — `html`, `css`, `nothing`
- `lit/decorators.js` — `customElement`, `property`
- `lit/directives/class-map.js` — `classMap`
- `lit/directives/style-map.js` — `styleMap`
- `@a2ui/web_core/data/model-processor` — `A2uiMessageProcessor`
- `@a2ui/web_core/types/primitives` — `Primitives.NumberValue`, `Primitives.StringValue`

### 저장소 내부 모듈
- [`./root.js`](./root.ts.md) — `Root` (기반 클래스)
- [`./styles.js`](./styles.ts.md) — `structuralStyles`
- [`./utils/utils.js`](./utils/utils.ts.md) — `extractNumberValue`, `extractStringValue`

## Exports

| 이름 | 종류 |
|---|---|
| `Slider` | 클래스 (`@customElement('a2ui-slider')`, `Root` 상속) |

## 상세 명세

### 클래스: `Slider`

`Root`를 상속하며 `@customElement('a2ui-slider')`로 등록된다.

#### 필드 및 프로퍼티

| 이름 | 타입 | 기본값 | 데코레이터 | 설명 |
|---|---|---|---|---|
| `value` | `Primitives.NumberValue \| null` | `null` | 슬라이더 현재 값. 리터럴 또는 바인딩 경로. `@property()` |
| `minValue` | `number` | `0` | 슬라이더 최솟값. `@property()` |
| `maxValue` | `number` | `0` | 슬라이더 최댓값. `@property()` |
| `label` | `Primitives.StringValue \| null` | `null` | 슬라이더 위에 표시할 레이블 텍스트. `@property()` |

#### `static styles`

`structuralStyles`와 인라인 `css` 블록을 배열로 결합한다.
- `* { box-sizing: border-box; }`
- `:host { display: block; flex: var(--weight); }`
- `input { display: block; width: 100%; }`
- `.description {}` — 빈 규칙(현재 스타일 없음)

---

#### `private #setBoundValue(value: string): void`

`path` 기반 바인딩에 값을 쓰는 메서드. 다음 조건 중 하나라도 해당하면 즉시 반환(no-op):
1. `this.value`가 falsy이거나 `this.processor`가 없음
2. `this.value`에 `'path'` 키가 없음
3. `this.value.path`가 falsy

모든 조건을 통과하면 `this.processor.setData(this.component, this.value.path, value, this.surfaceId ?? A2uiMessageProcessor.DEFAULT_SURFACE_ID)`를 호출하여 모델에 새 값을 저장한다.

---

#### `private #renderField(value: string | number): TemplateResult`

슬라이더 UI 구조를 반환한다.

1. 최외곽은 `<section class=${classMap(this.theme.components.Slider.container)}>` 래퍼.
2. `this.label`이 있으면 `extractStringValue(this.label, this.component, this.processor, this.surfaceId)`로 레이블 문자열을 가져와 `<label class=${...} for="data">` 렌더링. 없으면 `nothing`.
3. `<input type="range">`:
   - `autocomplete="off"`, `id="data"`, `name="data"`
   - `.value=${value}`, `min=${this.minValue ?? '0'}`, `max=${this.maxValue ?? '0'}`
   - `class=${classMap(this.theme.components.Slider.element)}`
   - `style`: `theme.additionalStyles?.Slider`가 있으면 `styleMap()`, 없으면 `nothing`
   - `@input` 핸들러: `evt.target`이 `HTMLInputElement`인지 확인 후 `#setBoundValue(evt.target.value)` 호출
4. `<span class=${classMap(this.theme.components.Slider.label)}>`: `this.value`가 있으면 `extractNumberValue(this.value, this.component, this.processor, this.surfaceId)`로 현재 숫자값 표시, 없으면 `'0'`.

---

#### `render(): TemplateResult | typeof nothing`

`this.value`가 객체인 경우 세 가지 경로로 분기:

1. `'literalNumber' in this.value && this.value.literalNumber` → `#renderField(this.value.literalNumber)` 반환.
2. `'literal' in this.value && this.value.literal !== undefined` → `#renderField(this.value.literal)` 반환.
3. `'path' in this.value && this.value.path`가 truthy:
   - `this.processor` 또는 `this.component`가 없으면 `` html`(no processor)` `` 반환.
   - `processor.getData(this.component, this.value.path, surfaceId)`로 값을 조회.
   - 결과가 `null`이면 `` html`Invalid value` `` 반환.
   - `string`도 `number`도 아니면 `` html`Invalid value` `` 반환.
   - 유효하면 `#renderField(textValue)` 반환.

세 경로에 모두 해당하지 않으면 `nothing`을 반환.

## 동작 흐름

`root.ts`가 `<a2ui-slider>`를 생성하고 `value`, `minValue`, `maxValue`, `label` 등을 설정한다. `render()`가 `value`의 종류를 판별해 실제 수치를 추출하여 `#renderField()`에 전달한다. `#renderField()`가 `<input type="range">`를 렌더링하고, 사용자가 드래그하면 `#setBoundValue()`로 모델 데이터를 갱신한다. signal 기반 상태 변경으로 `value`가 업데이트되면 `render()`가 재실행되어 슬라이더 위치와 표시 수치가 동기화된다.
