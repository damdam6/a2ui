# renderers/lit/src/0.8/ui/button.ts

## 개요

`Button` 클래스는 `a2ui-button` 커스텀 엘리먼트를 구현하는 Lit 웹 컴포넌트다. A2UI 컴포넌트 모델의 `Button` 노드를 렌더링하며, 클릭 시 `StateEvent<'a2ui.action'>` 이벤트를 디스패치해 상위 컴포넌트에 액션을 전달한다. 버튼 레이블은 슬롯(slot)을 통해 자식 콘텐츠로 제공된다.

## 의존성

### 외부 패키지
- `lit` — `html`, `css`, `nothing`
- `lit/decorators.js` — `customElement`, `property`
- `lit/directives/class-map.js` — `classMap`
- `lit/directives/style-map.js` — `styleMap`
- `@a2ui/web_core/types/types` — `Types` (네임스페이스)

### 저장소 내부 모듈
- [`./root.js`](./root.ts.md) — 베이스 클래스 `Root`
- [`../events/events.js`](../events/events.ts.md) — `StateEvent`
- [`./styles.js`](./styles.ts.md) — `structuralStyles`

## Exports

- `Button` (클래스, `@customElement('a2ui-button')` 데코레이터 적용)

## 상세 명세

### 클래스 `Button extends Root`

`Root`를 상속하는 Lit 커스텀 엘리먼트. `@customElement('a2ui-button')`로 등록된다.

#### 필드

| 필드 | 타입 | 기본값 | 설명 |
|------|------|--------|------|
| `action` | `Types.Action \| null` | `null` | `@property()` 데코레이터 적용. 클릭 시 디스패치할 액션 객체. |
| `primary` | `boolean \| null` | `false` | `@property()` 데코레이터 적용. 버튼의 primary 스타일 여부. |

#### 정적 필드 `styles`

`structuralStyles`와 인라인 CSS를 배열로 조합한다. 인라인 CSS 규칙:
- `:host` — `display: block`, `flex: var(--weight)`, `min-height: 0`

#### 메서드 `render(): TemplateResult`

`<button>` 요소를 반환한다.
- `class`에는 `classMap(this.theme.components.Button)`을 적용한다.
- `this.theme.additionalStyles?.Button`이 존재하면 `styleMap`으로 인라인 스타일을 적용하고, 없으면 `nothing`을 사용한다.
- `@click` 핸들러를 등록한다:
  1. `this.action`이 null이면 아무것도 하지 않고 반환한다.
  2. `StateEvent<'a2ui.action'>`를 다음 페이로드로 생성한다: `{ eventType: 'a2ui.action', action: this.action, dataContextPath: this.dataContextPath, sourceComponentId: this.id, sourceComponent: this.component }`.
  3. `this.dispatchEvent(evt)`로 이벤트를 발행한다. `StateEvent`는 `bubbles: true`, `cancelable: true`, `composed: true`로 설정된다.
- 내부에 `<slot></slot>`을 두어 외부에서 버튼 레이블 콘텐츠를 주입받는다.

## 동작 흐름

`action` 프로퍼티와 자식 컴포넌트(슬롯 콘텐츠)가 외부(`Root.renderComponentTree`)에서 바인딩되면 Lit이 `render()`를 호출해 테마 클래스가 적용된 `<button>` 요소를 생성한다. 사용자가 버튼을 클릭하면 `action`이 null이 아닌 경우에 한해 `a2uiaction` DOM 이벤트(버블링·composed)가 발행되고, 상위 Surface 등에서 이를 수신해 실제 액션 처리를 수행한다.
