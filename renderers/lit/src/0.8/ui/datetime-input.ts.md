# renderers/lit/src/0.8/ui/datetime-input.ts

## 개요

`DateTimeInput` 클래스는 `a2ui-datetimeinput` 커스텀 엘리먼트를 구현하는 Lit 웹 컴포넌트다. A2UI 컴포넌트 모델의 `DateTimeInput` 노드를 렌더링하며, `enableDate`/`enableTime` 플래그 조합에 따라 `datetime-local`, `date`, `time` 중 하나의 HTML input 타입을 사용한다. 문자열 값의 세 가지 형태(리터럴, 직접 리터럴, 데이터 경로 바인딩)를 지원하고, 브라우저 호환 날짜/시간 포맷으로 자동 변환한다.

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

- `DateTimeInput` (클래스, `@customElement('a2ui-datetimeinput')` 데코레이터 적용)

## 상세 명세

### 클래스 `DateTimeInput extends Root`

`Root`를 상속하는 Lit 커스텀 엘리먼트. `@customElement('a2ui-datetimeinput')`으로 등록된다.

#### 필드

| 필드 | 타입 | 기본값 | 데코레이터 옵션 | 설명 |
|------|------|--------|----------------|------|
| `value` | `Primitives.StringValue \| null` | `null` | `@property()` | 날짜/시간 값 객체 |
| `label` | `Primitives.StringValue \| null` | `null` | `@property()` | 레이블 값 객체 (현재 `#getPlaceholderText`로 대체됨) |
| `enableDate` | `boolean` | `true` | `@property({reflect: false, type: Boolean})` | 날짜 입력 활성화 여부 |
| `enableTime` | `boolean` | `true` | `@property({reflect: false, type: Boolean})` | 시간 입력 활성화 여부 |

#### 정적 필드 `styles`

`structuralStyles`와 인라인 CSS를 배열로 조합한다. 주요 규칙:
- `*` — `box-sizing: border-box`
- `:host` — `display: block`, `flex: var(--weight)`, `min-height: 0`, `overflow: auto`
- `input` — `display: block`, `border-radius: 8px`, `padding: 8px`, `border: 1px solid #ccc`, `width: 100%`

#### 비공개 메서드 `#setBoundValue(value: string): void`

데이터 경로 바인딩된 `value`를 모델에 쓰는 내부 헬퍼.

1. `this.value`가 없거나 `this.processor`가 없으면 즉시 반환한다.
2. `this.value`에 `'path'` 키가 없으면 반환한다.
3. `this.value.path`가 falsy이면 반환한다.
4. `this.processor.setData(this.component, this.value.path, value, surfaceId)`를 호출한다. `surfaceId`는 `this.surfaceId ?? A2uiMessageProcessor.DEFAULT_SURFACE_ID`를 사용한다.

#### 비공개 메서드 `#renderField(value: string): TemplateResult`

실제 날짜/시간 입력 UI를 구성하는 내부 렌더링 헬퍼.

- `<section>`에 `classMap(this.theme.components.DateTimeInput.container)`를 적용한다.
- `<label for="data">`에 `classMap(this.theme.components.DateTimeInput.label)`과 `#getPlaceholderText()`로 가져온 플레이스홀더 텍스트를 삽입한다.
- `<input>`을 렌더링한다:
  - `autocomplete="off"`, `id="data"`, `name="data"` 설정.
  - `class`는 `classMap(this.theme.components.DateTimeInput.element)`.
  - `style`은 `this.theme.additionalStyles?.DateTimeInput`이 있으면 `styleMap`, 없으면 `nothing`.
  - `@input` 핸들러: `evt.target`이 `HTMLInputElement`이면 `#setBoundValue(evt.target.value)` 호출.
  - `.value=${this.#formatInputValue(value)}` — Lit DOM 프로퍼티 바인딩으로 포맷된 값 설정.
  - `.placeholder=${this.#getPlaceholderText()}` — 플레이스홀더 설정.
  - `.type=${this.#getInputType()}` — input 타입 설정.

#### 비공개 메서드 `#getInputType(): string`

`enableDate`/`enableTime` 조합에 따라 input 타입 문자열을 반환한다.
- `enableDate && enableTime` → `'datetime-local'`
- `enableDate && !enableTime` → `'date'`
- `!enableDate && enableTime` → `'time'`
- 모두 false → `'datetime-local'` (폴백)

#### 비공개 메서드 `#formatInputValue(value: string): string`

브라우저 input 요소가 기대하는 포맷으로 날짜/시간 문자열을 변환한다.

1. `value`가 있으면 `new Date(value)`로 파싱한다. 없거나 `isNaN(date.getTime())`이면 빈 문자열을 반환한다.
2. `date.getFullYear()`, `getMonth() + 1`, `getDate()`, `getHours()`, `getMinutes()`를 각각 추출하고 `#padNumber`로 2자리 패딩한다.
3. `#getInputType()` 결과에 따라:
   - `'date'` → `YYYY-MM-DD` 형식 반환.
   - `'time'` → `HH:mm` 형식 반환.
   - 기타 → `YYYY-MM-DDTHH:mm` 형식 반환.
4. 주의: `toISOString()`은 UTC 기준이므로 사용하지 않고 로컬 시간 기준으로 직접 조합한다.

#### 비공개 메서드 `#padNumber(value: number): string`

`value.toString().padStart(2, '0')`으로 숫자를 최소 2자리 문자열로 패딩해 반환한다.

#### 비공개 메서드 `#getPlaceholderText(): string`

input 타입에 따라 플레이스홀더 텍스트를 반환한다.
- `'date'` → `'Date'`
- `'time'` → `'Time'`
- 기타 → `'Date & Time'`
- (TODO 주석: 이 값은 모델에서 전달받아야 함)

#### 메서드 `render(): TemplateResult | typeof nothing`

`this.value` 값의 형태에 따라 분기한다.
1. `'literalString'` 키가 있고 truthy이면 `#renderField(this.value.literalString)` 반환.
2. `'literal'` 키가 있고 `undefined`가 아니면 `#renderField(this.value.literal)` 반환.
3. `'path'` 키가 있고 truthy이면:
   - `processor` 또는 `component`가 없으면 `(no model)` 반환.
   - `processor.getData(...)`로 값을 조회해 문자열이 아니면 `(invalid)` 반환.
   - 정상 문자열이면 `#renderField(textValue)` 반환.
4. 위 조건에 해당하지 않으면 `nothing` 반환.

## 동작 흐름

외부에서 `value`, `enableDate`, `enableTime` 프로퍼티가 설정되면 `render()`가 `value` 타입을 판별해 `#renderField`를 호출한다. `#renderField`는 `#getInputType()`으로 input 종류를 결정하고, `#formatInputValue()`로 브라우저 호환 포맷으로 변환해 `<input>`에 바인딩한다. 사용자가 값을 변경하면 `@input` 핸들러가 `#setBoundValue`를 통해 모델 데이터를 업데이트한다.
