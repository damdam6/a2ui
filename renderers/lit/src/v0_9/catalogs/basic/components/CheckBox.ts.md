# renderers/lit/src/v0_9/catalogs/basic/components/CheckBox.ts

## 개요

`a2ui-checkbox` 커스텀 엘리먼트를 정의하는 파일이다. `BasicCatalogA2uiLitElement`를 상속하며 `CheckBoxApi` 스키마에 바인딩된다. `value`, `label`, `isValid`, `validationErrors`, `setValue` prop을 기반으로 체크박스 입력 UI를 렌더링한다. 유효성 검사 실패(`isValid === false`) 시 오류 색상 강조와 `validationErrors` 배열의 첫 번째 메시지를 인라인으로 표시한다. 카탈로그 등록용 `A2uiCheckBox` 상수도 export한다.

## 의존성

### 외부 패키지

- `lit` — `html`, `nothing`, `css`
- `lit/decorators.js` — `customElement`
- `lit/directives/class-map.js` — `classMap`
- `@a2ui/web_core/v0_9/basic_catalog` — `CheckBoxApi`
- `@a2ui/lit/v0_9` — `A2uiController`

### 저장소 내부 모듈

- [`../basic-catalog-a2ui-lit-element.js`](../basic-catalog-a2ui-lit-element.ts.md) — `BasicCatalogA2uiLitElement`

## Exports

| 이름 | 종류 |
|---|---|
| `A2uiCheckBoxElement` | 클래스 (커스텀 엘리먼트 `a2ui-checkbox`) |
| `A2uiCheckBox` | 상수 (카탈로그 등록용 객체) |

## 상세 명세

### 클래스: `A2uiCheckBoxElement`

- **선언:** `@customElement('a2ui-checkbox') export class A2uiCheckBoxElement extends BasicCatalogA2uiLitElement<typeof CheckBoxApi>`

#### 스타일 (`static styles`)

CSS 변수로 커스터마이징 가능한 항목:
- `--a2ui-checkbox-size`: 체크박스 크기 (기본: `1rem`)
- `--a2ui-checkbox-border-radius`: 모서리 반경 (기본: `4px`)
- `--a2ui-checkbox-gap`: 체크박스와 레이블 간격 (기본: `--a2ui-spacing-s, 0.5rem`)
- `--a2ui-checkbox-margin`: 컴포넌트 외부 마진 (기본: `--a2ui-spacing-m`)
- `--a2ui-checkbox-color-error`: 오류 상태 색상 (기본: `red`)
- `--a2ui-checkbox-label-font-size`: 레이블 글자 크기 (기본: `--a2ui-label-font-size, --a2ui-font-size-s`)
- `--a2ui-checkbox-label-font-weight`: 레이블 글자 굵기 (기본: `--a2ui-label-font-weight, bold`)

CSS 구조:
- `:host`: `display: block`
- `.container`: `flex`, `column` 방향, 외부 마진 적용
- `label.a2ui-checkbox`: inline-flex, 가운데 정렬, gap 및 폰트 스타일 적용, cursor pointer
- `label.invalid`: 오류 색상 적용
- `input`: 체크박스 크기, 배경, 테두리 설정
- `input.invalid`: `1px solid` 오류 색상의 `outline` 추가
- `.error`: 오류 메시지 — 오류 색상, `--a2ui-font-size-xs, 0.75rem` 크기, 상단 4px 마진

#### 메서드: `createController()`

- **시그니처:** `protected createController(): A2uiController<typeof CheckBoxApi>`
- `new A2uiController(this, CheckBoxApi)`를 반환한다.

#### 메서드: `render()`

- **시그니처:** `render(): TemplateResult | typeof nothing`
- `this.controller.props`가 없으면 `nothing`을 반환한다.
- `isInvalid = props.isValid === false`로 유효성 검사 실패 여부를 판단한다. (`isValid`가 `undefined`이면 유효한 상태)
- CSS 클래스 맵:
  - 레이블: `{ 'a2ui-checkbox': true, invalid: isInvalid }`
  - 입력: `{ invalid: isInvalid }`
- 렌더링 DOM 구조:
  - `.container` div
    - `<label class=${...}>` — 레이블 CSS 클래스 적용
      - `<input type="checkbox" class=${...}>` — Lit의 `.checked` 바인딩으로 DOM 프로퍼티 직접 설정(`props.value || false`). `@change` 이벤트에서 `(e.target as HTMLInputElement).checked` 값으로 `props.setValue?.(값)`을 호출해 상위 모델을 갱신한다.
      - `${props.label}` — 레이블 텍스트
    - `isInvalid && props.validationErrors?.length`가 참이면 `.error` div에 `props.validationErrors[0]` 표시. 그 외 `nothing`.

### 상수: `A2uiCheckBox`

`{ ...CheckBoxApi, tagName: 'a2ui-checkbox' }` — `CheckBoxApi`의 모든 속성에 `tagName: 'a2ui-checkbox'`를 추가한 카탈로그 등록용 객체.

## 동작 흐름

체크박스 상태 변경 시 `@change` 이벤트가 `props.setValue`를 호출해 상위 데이터 모델을 갱신한다. 모델 변경은 컨트롤러 구독을 통해 `props`를 업데이트하고 재렌더링을 트리거한다. 유효성 검사 실패(`isValid === false`)이면 레이블과 입력 요소에 `invalid` 클래스가 추가되어 오류 색상이 적용되고, `validationErrors` 배열의 첫 번째 메시지가 인라인으로 표시된다.
