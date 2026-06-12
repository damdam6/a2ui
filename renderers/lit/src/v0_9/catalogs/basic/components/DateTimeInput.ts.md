# renderers/lit/src/v0_9/catalogs/basic/components/DateTimeInput.ts

## 개요

`a2ui-datetimeinput` 커스텀 엘리먼트를 정의하는 파일이다. 날짜, 시간, 또는 날짜+시간 조합 입력 필드를 렌더링하며, `DateTimeInputApi` props에 따라 활성화되는 입력 유형을 결정한다. HTML5 date/time 입력 요소가 ISO 형식의 타임존 지시자나 초·밀리초를 포함한 값을 거부하는 문제를 해결하기 위해 `normalizeDateTimeValue` 함수로 값을 정규화한다.

## 의존성

### 외부 패키지
- `lit` — `html`, `nothing`, `css`
- `lit/decorators.js` — `customElement`
- `@a2ui/web_core/v0_9/basic_catalog` — `DateTimeInputApi`
- `@a2ui/lit/v0_9` — `A2uiController`

### 저장소 내부 모듈
- [`../basic-catalog-a2ui-lit-element.ts`](../basic-catalog-a2ui-lit-element.ts.md) — `BasicCatalogA2uiLitElement`

## Exports

- `A2uiDateTimeInputElement` — 클래스 (커스텀 엘리먼트)
- `A2uiDateTimeInput` — 상수 (객체: `DateTimeInputApi` 스프레드 + `tagName: 'a2ui-datetimeinput'`)

## 상세 명세

### `normalizeDateTimeValue(value: string | null | undefined, type: string): string`

HTML5 date/time 입력 요소가 수용하는 형식으로 값을 정규화하는 순수 함수.

**매개변수**:
- `value`: 입력 값. `null` 또는 `undefined`이면 빈 문자열 `''`을 즉시 반환한다.
- `type`: `'date'` | `'time'` | `'datetime-local'` 중 하나.

**동작**:
1. `value`가 falsy이면 `''`를 반환한다.
2. `'T'`가 포함되어 있으면 ISO 형식으로 간주하고 `split('T')`로 날짜 부분과 시간 부분으로 분리한다.
3. `datePart`: T가 있으면 `split[0]`의 앞 10글자, 없으면 `value` 자체의 앞 10글자 (없으면 `''`).
4. `timePart`: T가 있으면 `split[1]`의 앞 5글자(`HH:MM`), 없으면 `value` 자체의 앞 5글자 (없으면 `''`).
5. `type`에 따라:
   - `'date'` → `datePart` 반환 (형식: `YYYY-MM-DD`)
   - `'time'` → `timePart` 반환 (형식: `HH:MM`)
   - `'datetime-local'` → `` `${datePart}T${timePart}` `` 반환 (형식: `YYYY-MM-DDTHH:MM`)
6. 어떤 `type`도 매칭되지 않으면 `''`를 반환한다.

이 함수는 타임존 변환 없이 순수하게 문자열 조작으로만 동작한다.

### `A2uiDateTimeInputElement` 클래스

**데코레이터**: `@customElement('a2ui-datetimeinput')`

**상속**: `BasicCatalogA2uiLitElement<typeof DateTimeInputApi>`

#### 정적 필드

- `static styles`: `CSSResult` — `:host`를 `flex-direction: column`으로 배치, gap은 `--a2ui-spacing-xs`. `input`에는 배경색, 텍스트 색, 테두리, border-radius, padding을 CSS 변수로 설정. `label`에는 font-size와 font-weight를 CSS 변수로 설정.

#### `createController(): A2uiController`

`new A2uiController(this, DateTimeInputApi)`를 반환.

#### `render(): TemplateResult | typeof nothing`

1. `this.controller.props`가 없으면 `nothing` 반환.
2. `props.enableDate`와 `props.enableTime`이 모두 `false`(falsy)이면 `nothing` 반환.
3. `inputType` 결정:
   - `enableDate && enableTime` → `'datetime-local'`
   - `enableDate && !enableTime` → `'date'`
   - `!enableDate && enableTime` → `'time'`
4. `normalizeDateTimeValue(props.value, inputType)`으로 값을 정규화하여 `normalizedValue`에 저장.
5. 렌더링: `props.label`이 있으면 `<label>` 출력. `<input type={inputType} .value={normalizedValue}>` 출력. `@input` 이벤트에서 `props.setValue?.(e.target.value)`를 호출한다.

### `A2uiDateTimeInput` 상수

`DateTimeInputApi` 스프레드 + `tagName: 'a2ui-datetimeinput'`.

## 동작 흐름

props 수신 → 날짜/시간 활성화 여부 확인 → input 타입 결정 → ISO 값을 HTML5 호환 형식으로 정규화 → label과 input 렌더링. 사용자 입력은 `@input` 이벤트를 통해 즉시 `props.setValue`로 전파된다.
