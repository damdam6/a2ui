# renderers/lit/src/v0_9/catalogs/basic/components/Card.ts

## 개요

`a2ui-card` 커스텀 엘리먼트를 정의하는 파일이다. `BasicCatalogA2uiLitElement`를 상속하며 `CardApi` 스키마에 바인딩된다. 카드 컨테이너 역할을 하며 `child` prop으로 지정된 단일 자식 컴포넌트를 렌더링한다. 테두리, 모서리 반경, 패딩, 배경색, 그림자를 CSS 변수로 커스터마이징할 수 있다. 카탈로그 등록용 `A2uiCard` 상수도 export한다.

## 의존성

### 외부 패키지

- `lit` — `html`, `nothing`, `css`
- `lit/decorators.js` — `customElement`
- `@a2ui/web_core/v0_9/basic_catalog` — `CardApi`
- `@a2ui/lit/v0_9` — `A2uiController`

### 저장소 내부 모듈

- [`../basic-catalog-a2ui-lit-element.js`](../basic-catalog-a2ui-lit-element.ts.md) — `BasicCatalogA2uiLitElement`

## Exports

| 이름 | 종류 |
|---|---|
| `A2uiCardElement` | 클래스 (커스텀 엘리먼트 `a2ui-card`) |
| `A2uiCard` | 상수 (카탈로그 등록용 객체) |

## 상세 명세

### 클래스: `A2uiCardElement`

- **선언:** `@customElement('a2ui-card') export class A2uiCardElement extends BasicCatalogA2uiLitElement<typeof CardApi>`

#### 스타일 (`static styles`)

`:host`에 모든 스타일이 적용되며 CSS 변수로 커스터마이징 가능한 항목:
- `--a2ui-card-border`: 테두리 (기본: `1px solid #ccc`)
- `--a2ui-card-border-radius`: 모서리 반경 (기본: `--a2ui-border-radius, 8px`)
- `--a2ui-card-padding`: 내부 패딩 (기본: `--a2ui-spacing-m, 16px`)
- `--a2ui-card-background`: 배경색 (기본: `--a2ui-color-surface, #fff`)
- `--a2ui-card-box-shadow`: 그림자 (기본: `0 2px 4px rgba(0,0,0,0.1)`)
- `--a2ui-card-margin`: 외부 마진 (기본: `--a2ui-spacing-m`)
- 텍스트 색상: `var(--a2ui-color-on-surface, #333)` (변수 지정 없이 직접 참조)

#### 메서드: `createController()`

- **시그니처:** `protected createController(): A2uiController<typeof CardApi>`
- `new A2uiController(this, CardApi)`를 반환한다.

#### 메서드: `render()`

- **시그니처:** `render(): TemplateResult | typeof nothing`
- `this.controller.props`가 없으면 `nothing`을 반환한다.
- `props.child`가 있으면 `this.renderNode(props.child)`를 호출해 자식 컴포넌트를 렌더링한다. 없으면 빈 템플릿(공백)을 렌더링한다.

### 상수: `A2uiCard`

`{ ...CardApi, tagName: 'a2ui-card' }` — `CardApi`의 모든 속성에 `tagName: 'a2ui-card'`를 추가한 카탈로그 등록용 객체.

## 동작 흐름

컨텍스트가 설정되면 `CardApi` 스키마로 바인딩된 컨트롤러가 생성된다. `render()`에서 `props.child`가 있으면 `renderNode`를 통해 자식 컴포넌트를 재귀 렌더링하고, 없으면 빈 카드 컨테이너를 표시한다. 카드의 시각적 스타일은 모두 CSS 변수를 통해 외부에서 제어 가능하다.
