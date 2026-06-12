# renderers/lit/src/v0_9/catalogs/basic/basic-catalog-a2ui-lit-element.ts

## 개요

basic 카탈로그 컴포넌트들이 공통으로 상속하는 중간 추상 클래스 `BasicCatalogA2uiLitElement<Api>`를 정의한다. `A2uiLitElement`를 확장하며, DOM 연결 시 basic 카탈로그 공통 CSS를 문서에 주입하고, 업데이트마다 `weight` prop에 따른 flex 스타일과 테마의 `primaryColor`에 따른 CSS 색상 변수 4종을 호스트 엘리먼트에 동적으로 설정한다. 타입 유틸리티인 `ResolvedChildRef`와 `ResolvedChildList`도 export한다.

## 의존성

### 외부 패키지

- `@a2ui/web_core/v0_9` — `ComponentApi`, `ComponentId` (타입)
- `@a2ui/web_core/v0_9/basic_catalog` — `injectBasicCatalogStyles`, `computeColorVariant`

### 저장소 내부 모듈

- [`../../a2ui-lit-element.js`](../../a2ui-lit-element.ts.md) — `A2uiLitElement`

## Exports

| 이름 | 종류 |
|---|---|
| `ResolvedChildRef` | 타입 |
| `ResolvedChildList` | 타입 |
| `BasicCatalogA2uiLitElement<Api>` | 추상 클래스 |

## 상세 명세

### 타입: `ResolvedChildRef`

`type ResolvedChildRef = ComponentId | { id: ComponentId; basePath: string }`

자식 컴포넌트 참조를 나타내는 유니온 타입. 단순 ID 문자열이거나 ID와 데이터 기준 경로를 담은 객체다.

### 타입: `ResolvedChildList`

`type ResolvedChildList = ResolvedChildRef[]`

`ResolvedChildRef` 배열 타입.

### 추상 클래스: `BasicCatalogA2uiLitElement<Api extends ComponentApi>`

`A2uiLitElement<Api>`를 상속하는 추상 클래스.

#### 메서드: `connectedCallback()`

- **시그니처:** `connectedCallback(): void`
- 웹컴포넌트/Lit 생명주기 — 엘리먼트가 DOM에 삽입될 때 호출된다.
- `super.connectedCallback()`을 먼저 호출한다.
- `injectBasicCatalogStyles()`를 호출해 basic 카탈로그의 공통 CSS를 문서에 주입한다. 멱등성은 `injectBasicCatalogStyles` 내부에서 보장된다.

#### 메서드: `willUpdate(changedProperties: Map<string, any>)`

- **시그니처:** `willUpdate(changedProperties: Map<string, any>): void`
- `super.willUpdate(changedProperties)`를 먼저 호출한다.
- **flex/weight 처리:**
  - `this.controller?.props`를 `any`로 캐스팅하여 `props.weight`를 확인한다.
  - `props.weight`가 `undefined`가 아니면 `this.style.flex = String(props.weight)`를 설정한다.
  - `props.weight`가 `undefined`이면 `this.style.removeProperty('flex')`를 호출해 flex 속성을 제거한다.
- **primaryColor CSS 변수 처리:**
  - `this.context?.theme?.primaryColor`가 truthy이면 아래 4개의 CSS 변수를 `this.style.setProperty`로 인라인 스타일에 설정한다:
    - `--a2ui-color-primary`: `primaryColor` 값 직접 설정
    - `--a2ui-color-primary-light`: `computeColorVariant('light', { colorVar: '--a2ui-color-primary' })` 반환값
    - `--a2ui-color-primary-dark`: `computeColorVariant('dark', { colorVar: '--a2ui-color-primary' })` 반환값
    - `--a2ui-color-primary-hover`: `computeColorVariant('hover', { darkVar: '--a2ui-color-primary-dark', lightVar: '--a2ui-color-primary-light' })` 반환값
  - `primaryColor`가 없으면 위 4개 CSS 변수를 모두 `this.style.removeProperty`로 제거한다.

## 동작 흐름

basic 카탈로그의 모든 구체 컴포넌트는 이 클래스를 상속함으로써 CSS 주입과 동적 스타일(flex, 색상 변수) 업데이트를 자동으로 처리받는다. 개별 컴포넌트는 `createController()`와 `render()`만 구현하면 된다. 색상 변수는 호스트 엘리먼트의 인라인 스타일로 설정되므로, Shadow DOM 내부 컴포넌트들이 CSS 변수를 통해 테마 색상을 상속받는다.
