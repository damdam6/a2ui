# renderers/lit/src/0.8/ui/column.ts

## 개요

`Column` 클래스는 `a2ui-column` 커스텀 엘리먼트를 구현하는 Lit 웹 컴포넌트다. A2UI 컴포넌트 모델의 `Column` 노드를 렌더링하며, 세로 방향 flex 레이아웃 컨테이너를 제공한다. `alignment`와 `distribution` 속성을 통해 자식 요소의 교차축 정렬과 주축 배분 방식을 CSS 호스트 속성 선택자로 제어한다.

## 의존성

### 외부 패키지
- `lit` — `html`, `css`, `nothing`
- `lit/decorators.js` — `customElement`, `property`
- `lit/directives/class-map.js` — `classMap`
- `lit/directives/style-map.js` — `styleMap`
- `@a2ui/web_core/types/types` — `Types` (네임스페이스)

### 저장소 내부 모듈
- [`./root.js`](./root.ts.md) — 베이스 클래스 `Root`
- [`./styles.js`](./styles.ts.md) — `structuralStyles`

## Exports

- `Column` (클래스, `@customElement('a2ui-column')` 데코레이터 적용)

## 상세 명세

### 클래스 `Column extends Root`

`Root`를 상속하는 Lit 커스텀 엘리먼트. `@customElement('a2ui-column')`로 등록된다.

#### 필드

| 필드 | 타입 | 기본값 | 데코레이터 옵션 | 설명 |
|------|------|--------|----------------|------|
| `alignment` | `Types.ResolvedColumn['alignment']` | `'stretch'` | `@property({reflect: true, type: String})` | 교차축 정렬. `'start'`, `'center'`, `'end'`, `'stretch'` 중 하나. |
| `distribution` | `Types.ResolvedColumn['distribution']` | `'start'` | `@property({reflect: true, type: String})` | 주축 배분. `'start'`, `'center'`, `'end'`, `'spaceBetween'`, `'spaceAround'`, `'spaceEvenly'` 중 하나. |

두 속성 모두 `reflect: true`로 설정되어 DOM 어트리뷰트와 프로퍼티가 양방향 동기화된다.

#### 정적 필드 `styles`

`structuralStyles`와 인라인 CSS를 배열로 조합한다. 주요 CSS 규칙:
- `*` — `box-sizing: border-box`
- `:host` — `display: flex`, `flex: var(--weight)`
- `section` — `display: flex`, `flex-direction: column`, `min-width: 100%`, `height: 100%`
- `:host([alignment='start']) section` — `align-items: start`
- `:host([alignment='center']) section` — `align-items: center`
- `:host([alignment='end']) section` — `align-items: end`
- `:host([alignment='stretch']) section` — `align-items: stretch`
- `:host([distribution='start']) section` — `justify-content: start`
- `:host([distribution='center']) section` — `justify-content: center`
- `:host([distribution='end']) section` — `justify-content: end`
- `:host([distribution='spaceBetween']) section` — `justify-content: space-between`
- `:host([distribution='spaceAround']) section` — `justify-content: space-around`
- `:host([distribution='spaceEvenly']) section` — `justify-content: space-evenly`

#### 메서드 `render(): TemplateResult`

`<section>` 요소를 반환한다.
- `class`에는 `classMap(this.theme.components.Column)`을 적용한다.
- `this.theme.additionalStyles?.Column`이 존재하면 `styleMap`으로 인라인 스타일을 적용하고, 없으면 `nothing`을 사용한다.
- 내부에 `<slot></slot>`을 두어 자식 컴포넌트를 Light DOM으로 투영받는다.

## 동작 흐름

`alignment`와 `distribution` 프로퍼티가 설정되면 `reflect: true` 옵션에 의해 호스트 엘리먼트의 어트리뷰트로 반영된다. Shadow DOM CSS의 `:host([alignment='...'])` 선택자가 해당 어트리뷰트 값에 따라 `<section>`의 `align-items` 및 `justify-content`를 자동으로 변경한다. 이로써 JavaScript 없이 CSS 선택자만으로 레이아웃이 제어된다.
