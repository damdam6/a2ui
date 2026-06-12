# renderers/lit/src/0.8/ui/card.ts

## 개요

`Card` 클래스는 `a2ui-card` 커스텀 엘리먼트를 구현하는 Lit 웹 컴포넌트다. A2UI 컴포넌트 모델의 `Card` 노드를 렌더링하며, 자식 콘텐츠를 슬롯으로 감싸는 컨테이너 역할을 한다. 추가 프로퍼티 없이 테마 클래스와 추가 스타일만 적용한 `<section>` 래퍼를 제공한다.

## 의존성

### 외부 패키지
- `lit` — `html`, `css`, `nothing`
- `lit/decorators.js` — `customElement`
- `lit/directives/class-map.js` — `classMap`
- `lit/directives/style-map.js` — `styleMap`

### 저장소 내부 모듈
- [`./root.js`](./root.ts.md) — 베이스 클래스 `Root`
- [`./styles.js`](./styles.ts.md) — `structuralStyles`

## Exports

- `Card` (클래스, `@customElement('a2ui-card')` 데코레이터 적용)

## 상세 명세

### 클래스 `Card extends Root`

`Root`를 상속하는 Lit 커스텀 엘리먼트. `@customElement('a2ui-card')`로 등록된다. 자체 `@property()` 필드를 추가로 선언하지 않으며, 모든 데이터 프로퍼티는 `Root`로부터 상속한다.

#### 정적 필드 `styles`

`structuralStyles`와 인라인 CSS를 배열로 조합한다. 인라인 CSS 규칙:
- `*` — `box-sizing: border-box`
- `:host` — `display: block`, `flex: var(--weight)`, `min-height: 0`, `overflow: auto`
- `section` — `height: 100%`, `width: 100%`, `min-height: 0`, `overflow: auto`
- `section ::slotted(*)` — `height: 100%`, `width: 100%` (슬롯에 삽입된 자식이 카드를 꽉 채우도록 함)

#### 메서드 `render(): TemplateResult`

`<section>` 요소를 반환한다.
- `class`에는 `classMap(this.theme.components.Card)`를 적용한다.
- `this.theme.additionalStyles?.Card`가 존재하면 `styleMap`으로 인라인 스타일을 적용하고, 없으면 `nothing`을 사용한다.
- 내부에 `<slot></slot>`을 두어 자식 컴포넌트를 Light DOM으로 투영받는다.

## 동작 흐름

`Root`의 `willUpdate`가 `childComponents` 변경을 감지하면 effect를 통해 자식 컴포넌트 트리가 Light DOM에 렌더링되고, `Card`의 `<slot>`이 이를 투영한다. `Card` 자체는 별도 상태 없이 테마 기반 CSS 클래스와 추가 스타일을 `<section>` 컨테이너에 적용하는 역할만 담당한다.
