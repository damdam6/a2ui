# renderers/lit/src/v0_9/catalogs/basic/components/ChoicePicker.ts

## 개요

`a2ui-choicepicker` 커스텀 엘리먼트를 정의하는 파일이다. 단일 선택 또는 다중 선택을 지원하는 선택 UI를 렌더링하며, `chips` 스타일과 `radio`/`checkbox` 스타일 두 가지 표시 방식을 제공한다. 선택 항목 필터링 기능도 선택적으로 활성화할 수 있다. `ChoicePickerApi`를 기반으로 컨트롤러를 생성하고, props를 통해 렌더링 동작을 제어한다.

## 의존성

### 외부 패키지
- `lit` — `html`, `nothing`, `css`
- `lit/decorators.js` — `customElement`, `state`
- `lit/directives/class-map.js` — `classMap`
- `@a2ui/web_core/v0_9/basic_catalog` — `ChoicePickerApi`
- `@a2ui/lit/v0_9` — `A2uiController`

### 저장소 내부 모듈
- [`../basic-catalog-a2ui-lit-element.ts`](../basic-catalog-a2ui-lit-element.ts.md) — `BasicCatalogA2uiLitElement`

## Exports

- `A2uiChoicePickerElement` — 클래스 (`LitElement` 기반 커스텀 엘리먼트)
- `A2uiChoicePicker` — 상수 (객체: `ChoicePickerApi`를 스프레드한 뒤 `tagName: 'a2ui-choicepicker'`를 추가)

## 상세 명세

### `A2uiChoicePickerElement` 클래스

**데코레이터**: `@customElement('a2ui-choicepicker')`

**상속**: `BasicCatalogA2uiLitElement<typeof ChoicePickerApi>`

#### 정적 필드

- `static styles`: `CSSResult` — Shadow DOM 내 스타일을 정의한다. 주요 내용:
  - `:host`는 `flex-direction: column`으로 세로 배치하며 `--a2ui-choicepicker-gap` 변수로 간격을 제어한다.
  - `.options`는 `flex-direction: column`, `.chips`와 함께 사용 시 `flex-direction: row`로 전환되며 `flex-wrap: wrap`이 적용된다.
  - `label`은 색상과 폰트 크기를 CSS 변수로 제어한다. `:host > label`에만 굵기가 적용된다.
  - `.filter-input`은 텍스트 입력 필드 스타일로 포커스 시 `border-color`가 primary 색으로 바뀐다.
  - `.chip`은 pill 형태 버튼으로 기본 배경은 surface 색, `.chip.selected`가 되면 배경이 primary 색으로 변한다.

#### 인스턴스 필드

- `@state() accessor filter: string` — 기본값 `''`. 필터 입력 필드의 현재 텍스트를 저장하며, 변경 시 자동으로 재렌더링을 트리거한다.

#### `createController(): A2uiController`

반환: `new A2uiController(this, ChoicePickerApi)`. 부모 클래스가 호출하는 팩토리 메서드로, 이 컴포넌트가 사용할 컨트롤러 인스턴스를 생성한다.

#### `render(): TemplateResult | typeof nothing`

1. `this.controller.props`가 없으면 `nothing`을 반환한다.
2. `props.value`가 배열이면 그대로 사용, 아니면 빈 배열 `[]`로 초기화하여 `selected`에 저장한다.
3. `props.variant === 'multipleSelection'`이면 `isMulti = true`.
4. `props.displayStyle === 'chips'`이면 `isChips = true`.
5. 내부 클로저 `toggle(val: string)`:
   - `props.setValue`가 없으면 즉시 반환한다.
   - `isMulti`가 `true`이면: `selected`에 `val`이 있으면 해당 값을 제거한 배열, 없으면 추가한 배열을 `props.setValue`에 전달한다.
   - `isMulti`가 `false`이면: `props.setValue([val])`로 단일 값 배열을 전달한다.
6. `props.options`(없으면 `[]`)를 필터링한다:
   - `props.filterable`이 `false`이거나 `this.filter`가 빈 문자열이면 모든 옵션을 포함한다.
   - `props.filterable`이 `true`이고 `this.filter`가 있으면 `opt.label`을 소문자로 변환 후 `this.filter`의 소문자 변환 값을 포함하는 옵션만 남긴다.
7. 렌더링:
   - `props.label`이 있으면 `<label>` 태그로 출력한다.
   - `props.filterable`이 `true`이면 `type="text"` 입력 필드를 출력하며, `@input` 이벤트에서 `this.filter`를 갱신한다.
   - 옵션 목록 컨테이너: `isChips`가 `true`이면 클래스 `options chips`, 아니면 `options`만 적용한다.
   - `isChips`가 `true`이면 각 옵션을 `<button class="chip [selected]">`으로 렌더링하고, `aria-pressed`를 선택 여부로 설정하며 `@click`에서 `toggle`을 호출한다.
   - `isChips`가 `false`이면 `<label><input type="checkbox|radio">...</label>` 형태로 렌더링하며 `type`은 `isMulti` 여부로 결정하고 `@change`에서 `toggle`을 호출한다.

### `A2uiChoicePicker` 상수

`ChoicePickerApi`의 모든 필드를 스프레드하고 `tagName: 'a2ui-choicepicker'`를 추가한 객체. 외부에서 이 컴포넌트를 등록·참조할 때 사용한다.

## 동작 흐름

컨트롤러로부터 props를 수신 → 선택 상태·표시 방식·필터 조건 계산 → 선택 항목 필터링 → 선택 방식(chips / radio·checkbox)에 따라 적절한 HTML 구조를 렌더링. 사용자 입력(클릭, 체크 변경)은 `toggle` 클로저를 통해 `props.setValue`로 전파되며, 필터 입력은 `@state` 변수 `filter`를 업데이트하여 Lit의 반응성 시스템을 통해 재렌더링을 유발한다.
