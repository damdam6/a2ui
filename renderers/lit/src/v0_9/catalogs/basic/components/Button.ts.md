# renderers/lit/src/v0_9/catalogs/basic/components/Button.ts

## 개요

`a2ui-basic-button` 커스텀 엘리먼트를 정의하는 파일이다. `BasicCatalogA2uiLitElement`를 상속하며 `ButtonApi` 스키마에 바인딩된다. `variant`, `isValid`, `action`, `child` prop을 기반으로 버튼을 렌더링하며, primary/default/borderless 세 가지 외형 변형을 CSS 변수로 커스터마이징할 수 있다. 카탈로그 등록용 `A2uiButton` 상수도 export한다.

## 의존성

### 외부 패키지

- `lit` — `html`, `nothing`, `css`
- `lit/decorators.js` — `customElement`
- `lit/directives/class-map.js` — `classMap`
- `@a2ui/web_core/v0_9/basic_catalog` — `ButtonApi`
- `@a2ui/lit/v0_9` — `A2uiController`

### 저장소 내부 모듈

- [`../basic-catalog-a2ui-lit-element.js`](../basic-catalog-a2ui-lit-element.ts.md) — `BasicCatalogA2uiLitElement`

## Exports

| 이름 | 종류 |
|---|---|
| `A2uiBasicButtonElement` | 클래스 (커스텀 엘리먼트 `a2ui-basic-button`) |
| `A2uiButton` | 상수 (카탈로그 등록용 객체) |

## 상세 명세

### 클래스: `A2uiBasicButtonElement`

- **선언:** `@customElement('a2ui-basic-button') export class A2uiBasicButtonElement extends BasicCatalogA2uiLitElement<typeof ButtonApi>`

#### 스타일 (`static styles`)

CSS 변수로 커스터마이징 가능한 항목:
- `--a2ui-color-primary`: primary 변형 배경색 (기본 `#17e`)
- `--a2ui-color-on-primary`: primary 변형 텍스트색
- `--a2ui-color-secondary`, `--a2ui-color-on-secondary`: default 변형 색상 (텍스트 기본 `#333`)
- `--a2ui-button-border`: 테두리 (기본: `1px solid #ccc`)
- `--a2ui-button-border-radius`: 모서리 반경 (기본: `--a2ui-spacing-s, 0.25rem`)
- `--a2ui-button-padding`: 패딩 (기본: `--a2ui-spacing-m --a2ui-spacing-l` = `0.5rem 1rem`)
- `--a2ui-button-margin`: `:host` 외부 마진 (기본: `--a2ui-spacing-m`)

CSS 클래스 구조 (Lit Shadow DOM 내부):
- `.a2ui-button`: 기본 스타일 — 배경 `--a2ui-color-surface, #fff`, 테두리, inline-flex, cursor pointer
- `.a2ui-button.a2ui-button-primary`: 배경색 `--_color-primary`, 텍스트색 `--a2ui-color-on-primary, #fff`
- `.a2ui-button:hover`: 배경 `--a2ui-color-secondary-hover, #ddd`
- `.a2ui-button.a2ui-button-primary:hover`: 배경 `--a2ui-color-primary-hover, #fbd`
- `.a2ui-button.a2ui-button-borderless`: 배경 없음, 패딩 0, 텍스트색 primary

#### 메서드: `createController()`

- **시그니처:** `protected createController(): A2uiController<typeof ButtonApi>`
- `new A2uiController(this, ButtonApi)`를 반환한다.

#### 메서드: `render()`

- **시그니처:** `render(): TemplateResult | typeof nothing`
- `this.controller.props`가 없으면 `nothing`을 반환한다.
- `isDisabled`를 `props.isValid === false`로 판단한다. (`isValid`가 `undefined`이면 disabled가 아님)
- CSS 클래스 맵을 구성한다: `'a2ui-button': true`, `'a2ui-button-' + (props.variant || 'default'): true`. `variant`가 없으면 `'a2ui-button-default'` 클래스가 적용된다.
- `<button>` 엘리먼트를 렌더링한다:
  - `class=${classMap(classes)}`로 클래스 적용
  - `@click` 핸들러: `isDisabled`가 `false`이고 `props.action`이 있으면 `props.action()` 호출. disabled 상태이거나 action이 없으면 아무 동작 없음
  - `?disabled=${isDisabled}`로 HTML disabled 속성 설정
  - 자식: `props.child`가 있으면 `this.renderNode(props.child)`, 없으면 `nothing`

### 상수: `A2uiButton`

`{ ...ButtonApi, tagName: 'a2ui-basic-button' }` — `ButtonApi`의 모든 속성에 `tagName: 'a2ui-basic-button'`을 추가한 카탈로그 등록용 객체.

## 동작 흐름

컨텍스트가 설정되면 `ButtonApi` 스키마로 바인딩된 컨트롤러가 생성된다. `render()`에서 variant에 따라 CSS 클래스를 결정하고, `isValid === false`이면 버튼을 비활성화한다. 자식 컴포넌트는 `renderNode`를 통해 재귀적으로 렌더링된다. 클릭 이벤트는 disabled 상태가 아닐 때만 `props.action`을 호출한다.
