# renderers/lit/src/v0_9/catalogs/basic/components/TextField.ts

## 개요

`a2ui-basic-textfield` 커스텀 엘리먼트를 정의하는 파일로, A2UI Basic Catalog의 텍스트 입력 컴포넌트를 Lit 웹 컴포넌트로 구현한다. 단일 줄 입력(`<input>`), 여러 줄 텍스트 영역(`<textarea>`), 숫자 입력, 비밀번호 입력을 `variant` 프로퍼티에 따라 조건부로 렌더링한다. 유효성 검사 실패 시 경계선 색상 변경과 오류 메시지 표시를 지원한다. 카탈로그 등록용 `A2uiTextField` 상수도 export한다.

## 의존성

### 외부 패키지
- `lit`: `html`, `nothing`, `css`
- `lit/decorators.js`: `customElement`
- `lit/directives/class-map.js`: `classMap`
- `@a2ui/web_core/v0_9/basic_catalog`: `TextFieldApi`
- `@a2ui/lit/v0_9`: `A2uiController`

### 저장소 내부 모듈
- [`../basic-catalog-a2ui-lit-element.js`](../basic-catalog-a2ui-lit-element.ts.md) — `BasicCatalogA2uiLitElement` 베이스 클래스 제공

## Exports

- `A2uiBasicTextFieldElement` (클래스): `a2ui-basic-textfield` 커스텀 엘리먼트 구현체
- `A2uiTextField` (상수 객체): `TextFieldApi`에 `tagName: 'a2ui-basic-textfield'`를 병합한 카탈로그 등록용 API 객체

## 상세 명세

### `A2uiBasicTextFieldElement` 클래스

`BasicCatalogA2uiLitElement<typeof TextFieldApi>`를 상속하는 Lit 커스텀 엘리먼트. `@customElement('a2ui-basic-textfield')` 데코레이터로 등록된다.

#### `static styles`
컴포넌트 전용 CSS 정의. `:host`는 `display: flex; flex-direction: column; gap: var(--a2ui-spacing-xs, 0.25rem)` 레이아웃을 가진다. 아래 CSS 커스텀 프로퍼티를 통해 외부에서 스타일 오버라이드가 가능하다:
- `--a2ui-textfield-border`: 테두리 스타일 (기본값: `--a2ui-border`)
- `--a2ui-textfield-border-radius`: 테두리 반경 (기본값: `--a2ui-spacing-m`)
- `--a2ui-textfield-padding`: 내부 여백 (기본값: `--a2ui-spacing-m`)
- `--a2ui-textfield-color-border-focus`: 포커스 시 테두리 색상 (기본값: `--a2ui-color-primary`, 폴백: `#17e`)
- `--a2ui-textfield-color-error`: 유효성 오류 색상 (기본값: `red`)
- `--a2ui-textfield-label-font-size`: 레이블 폰트 크기 (기본값: `--a2ui-label-font-size` → `--a2ui-font-size-s`)
- `--a2ui-textfield-label-font-weight`: 레이블 폰트 굵기 (기본값: `--a2ui-label-font-weight` → `bold`)
- 글로벌 입력 변수: `--a2ui-color-input` (배경색), `--a2ui-color-on-input` (텍스트 색상)

#### `protected createController(): A2uiController`
- 시그니처: `createController(): A2uiController`
- `new A2uiController(this, TextFieldApi)`를 생성하여 반환한다. 베이스 클래스의 추상 메서드를 구현한다.

#### `render(): TemplateResult | typeof nothing`
- 시그니처: `render(): TemplateResult | typeof nothing`
- `this.controller.props`가 없으면 `nothing`을 반환한다.
- `props.isValid === false`이면 `isInvalid = true`로 설정한다.
- `@input` 이벤트 핸들러 `onInput`을 정의하여 `(e.target as HTMLInputElement).value`를 `props.setValue?.()`에 전달한다.
- `type` 변수를 `'text'`로 초기화한 뒤 `props.variant`에 따라 변환한다:
  - `'number'` → `type = 'number'`
  - `'obscured'` → `type = 'password'`
- `classMap`을 사용하여 `a2ui-textfield` 클래스와 조건부 `invalid` 클래스를 적용한다.
- `props.label`이 있으면 `<label>` 엘리먼트를 렌더링한다.
- `props.variant === 'longText'`이면 `<textarea>`를, 그 외 모든 variant에는 `<input type=${type}>`를 렌더링한다. 두 경우 모두 `.value=${props.value || ''}`으로 단방향 바인딩하고 `@input=${onInput}` 이벤트를 등록한다.
- `isInvalid`가 참이고 `props.validationErrors` 배열에 항목이 있으면 첫 번째 오류 메시지를 `class="error"` div로 렌더링한다.

### `A2uiTextField` 상수 객체

`TextFieldApi`의 모든 프로퍼티를 스프레드(`...TextFieldApi`)하고 `tagName: 'a2ui-basic-textfield'`를 추가한 평범한 객체 리터럴. 카탈로그에 컴포넌트를 등록할 때 사용된다.

## 동작 흐름

`A2uiController`가 `TextFieldApi` 스키마에 따라 컴포넌트 모델에서 `props`를 가져온다. `render()`는 `props`를 읽어 적절한 입력 엘리먼트를 선택하고, 사용자 입력 이벤트가 발생하면 `props.setValue`를 통해 데이터 모델을 업데이트한다. 유효성 상태(`isValid`, `validationErrors`)는 CSS 클래스와 오류 텍스트 표시에 반영된다.
