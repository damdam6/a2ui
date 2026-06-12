# renderers/angular/src/v0_9/catalog/basic/text-field.component.ts

## 개요

A2UI v0.9 TextField 컴포넌트의 Angular 구현체다. 선택적 레이블과 텍스트 입력 필드를 렌더링하며, `variant` 프로퍼티에 따라 일반 텍스트(`text`), 비밀번호(`password`), 숫자(`number`) 입력 타입을 지원한다. 사용자 입력 시 `onUpdate` 콜백으로 데이터 모델을 즉시 갱신하며, 검증 오류 메시지 목록도 표시한다.

## 의존성

### 외부 패키지
- `@angular/core`: `Component`, `computed`, `ChangeDetectionStrategy`
- `@a2ui/web_core/v0_9/basic_catalog`: `TextFieldApi`

### 저장소 내부 모듈
- [`./basic-catalog-component`](./basic-catalog-component.ts.md): 기반 추상 클래스

## Exports

- `TextFieldComponent` — Angular standalone 컴포넌트 클래스

## 상세 명세

### `TextFieldComponent`

**데코레이터**: `@Component`
- `selector`: `'a2ui-v09-text-field'`
- `standalone: true`
- `imports`: `[]`
- `changeDetection`: `ChangeDetectionStrategy.OnPush`

**상속**: `BasicCatalogComponent<typeof TextFieldApi>`

**computed 필드**

| 이름 | 타입 | 소스 | 설명 |
|------|------|------|------|
| `label` | `Signal<string \| undefined>` | `props()['label']?.value()` | 레이블 텍스트. falsy이면 레이블 엘리먼트 미렌더링 |
| `value` | `Signal<string>` | `props()['value']?.value() \|\| ''` | 입력 필드 현재 값. 없으면 빈 문자열 |
| `variant` | `Signal<string \| undefined>` | `props()['variant']?.value()` | 입력 타입 결정에 사용 |
| `inputType` | `Signal<'text' \| 'password' \| 'number'>` | `variant()` switch 결과 | 아래 로직 참조 |

#### `inputType` 결정 로직
`variant()` 값에 따라 switch:
- `'obscured'` → `'password'`
- `'number'` → `'number'`
- 그 외(기본 포함) → `'text'`

**메서드**

#### `handleInput(event: Event): void`
`(event.target as HTMLInputElement).value`를 문자열로 읽어 `this.props()['value']?.onUpdate(value)`를 호출한다. 이를 통해 해당 데이터 경로를 구독하는 모든 리스너에게 변경이 전파된다.

**템플릿 구조**

1. `div.a2ui-text-field-container`(flex column) 안에 레이블, 입력 필드, 오류 메시지를 배치한다.
2. `label()` 값이 truthy이면 `<label>` 엘리먼트를 렌더링한다.
3. `<input>`에 `[type]`, `[value]`, `(input)` 이벤트를 바인딩한다. `props()['isValid']?.value() === false`이면 `invalid` CSS 클래스가 추가된다.
4. `props()['validationErrors']?.value()` 배열을 `@for`로 순회하며, 각 메시지에 `div.a2ui-error-message`를 렌더링한다.

**스타일 (CSS 변수 지원)**

| CSS 변수 | 기본값 | 역할 |
|----------|--------|------|
| `--a2ui-textfield-label-font-size` | `var(--a2ui-label-font-size, var(--a2ui-font-size-s, 14px))` | 레이블 폰트 크기 |
| `--a2ui-textfield-label-font-weight` | `bold` | 레이블 폰트 굵기 |
| `--a2ui-textfield-padding` | `8px` | 입력 필드 패딩 |
| `--a2ui-textfield-border` | `1px solid var(--a2ui-color-border, #ccc)` | 입력 필드 테두리 |
| `--a2ui-textfield-border-radius` | `4px` | 입력 필드 테두리 반지름 |
| `--a2ui-color-input` | `#fff` | 입력 필드 배경색 |
| `--a2ui-color-on-input` | `#333` | 입력 필드 텍스트 색상 |
| `--a2ui-textfield-color-border-focus` | `var(--a2ui-color-primary, #17e)` | 포커스 시 테두리 색상 |
| `--a2ui-textfield-color-error` | `var(--a2ui-color-error, red)` | 오류 상태 테두리 및 오류 메시지 텍스트 색상 |

`input.invalid` 클래스가 적용되면 테두리 색이 `--a2ui-textfield-color-error`로 변경된다.

## 동작 흐름

컴포넌트는 `props()`에서 `label`, `value`, `variant`를 computed signal로 읽는다. 렌더링 시 `label`이 truthy이면 레이블을 보여주고, `inputType` computed가 `variant`를 기반으로 HTML `type` 속성을 결정한다. 사용자가 입력할 때마다 `handleInput`이 문자열 값을 `onUpdate`로 전달한다. 검증 상태는 `isValid` 및 `validationErrors` 프로퍼티를 통해 외부에서 주입되며, `isValid`가 `false`이면 `invalid` 클래스가 추가되고 `validationErrors` 배열의 각 메시지가 오류 div로 렌더링된다.
