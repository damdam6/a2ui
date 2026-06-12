# renderers/lit/src/0.8/ui/divider.ts

## 개요

`<a2ui-divider>` Web Component를 정의하는 파일이다. 수평선(`<hr>`) 하나를 렌더링하며, 테마 시스템을 통해 CSS 클래스와 추가 인라인 스타일을 적용할 수 있다. `Root` 기반 클래스를 상속하므로 A2UI의 공통 속성(theme, processor, surfaceId 등)을 모두 갖는다.

## 의존성

### 외부 패키지
- `lit` — `html`, `css`, `nothing`
- `lit/decorators.js` — `customElement`
- `lit/directives/style-map.js` — `styleMap`
- `lit/directives/class-map.js` — `classMap`

### 저장소 내부 모듈
- [`./root.js`](../root.ts.md) — 베이스 클래스 `Root`
- [`./styles.js`](../styles.ts.md) — `structuralStyles` (공통 구조 스타일)

## Exports

| 이름 | 종류 | 설명 |
|---|---|---|
| `Divider` | 클래스 (LitElement 서브클래스) | `<a2ui-divider>` 커스텀 엘리먼트 |

## 상세 명세

### `@customElement('a2ui-divider') class Divider extends Root`

**Static 필드**

- `static styles: CSSResultGroup` — `structuralStyles`와 컴포넌트 로컬 CSS로 구성된 배열.
  - `:host`는 `display: block`, `min-height: 0`, `overflow: auto`.
  - `hr`는 `height: 1px`, `background: #ccc`, `border: none`.

**인스턴스 필드**

`Root`로부터 상속받는 필드(`theme`, `processor`, `surfaceId`, `component` 등)를 모두 가지며, `Divider`가 직접 선언하는 추가 `@property`는 없다.

---

#### `render(): TemplateResult`

1. `<hr>` 엘리먼트 하나를 반환한다.
2. `class` 속성은 `classMap(this.theme.components.Divider)`로 지정해 테마에서 정의된 CSS 클래스를 적용한다.
3. `style` 속성은 `this.theme.additionalStyles?.Divider`가 존재하면 `styleMap(...)`, 없으면 `nothing`을 사용한다.

## 동작 흐름

컴포넌트가 DOM에 연결되면 테마 컨텍스트를 소비해 `theme`를 얻는다 (Root의 `@consume` 처리). `render`가 호출될 때 테마의 Divider 컴포넌트 클래스맵과 추가 스타일맵을 조합해 `<hr>`을 렌더링한다. 상태나 이벤트 처리가 없어 순수 표현 컴포넌트다.
