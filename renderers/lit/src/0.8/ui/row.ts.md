# renderers/lit/src/0.8/ui/row.ts

## 개요

`Row`는 `a2ui-row` 커스텀 엘리먼트로, 자식 컴포넌트를 수평 방향으로 배치하는 컨테이너 역할을 한다. `Root`를 상속하여 A2UI 컴포넌트 트리 내에서 동작하며, `alignment`(교차축 정렬)와 `distribution`(주축 배분) 두 속성을 CSS 반영(reflect) 속성으로 노출해 Shadow DOM의 `:host([attr])` 선택자 기반 스타일링에 활용한다. 테마 컨텍스트에서 CSS 클래스와 추가 스타일을 읽어 렌더링한다.

## 의존성

### 외부 패키지
- `lit` — `html`, `css`, `nothing`
- `lit/decorators.js` — `customElement`, `property`
- `lit/directives/class-map.js` — `classMap`
- `lit/directives/style-map.js` — `styleMap`
- `@a2ui/web_core/types/types` — `Types` 네임스페이스(`Types.ResolvedRow` 타입 사용)

### 저장소 내부 모듈
- [`./root.js`](./root.ts.md) — `Root` (기반 클래스)
- [`./styles.js`](./styles.ts.md) — `structuralStyles`

## Exports

| 이름 | 종류 |
|---|---|
| `Row` | 클래스 (`@customElement('a2ui-row')`, `Root` 상속) |

## 상세 명세

### 클래스: `Row`

`Root`를 상속하며 `@customElement('a2ui-row')`로 등록된다.

#### 필드 및 프로퍼티

| 이름 | 타입 | 기본값 | 데코레이터 | 설명 |
|---|---|---|---|---|
| `alignment` | `Types.ResolvedRow['alignment']` | `'stretch'` | `@property({reflect: true, type: String})` | 교차축(cross-axis) 자식 정렬 방식 |
| `distribution` | `Types.ResolvedRow['distribution']` | `'start'` | `@property({reflect: true, type: String})` | 주축(main-axis) 자식 배분 방식 |

두 프로퍼티 모두 `reflect: true`이므로, TypeScript에서 값이 바뀌면 HTML 어트리뷰트로도 반영된다. 이를 통해 아래 `static styles`의 `:host([alignment='...'])`, `:host([distribution='...'])` 선택자가 동작한다.

허용 `alignment` 값: `'start'`, `'center'`, `'end'`, `'stretch'`  
허용 `distribution` 값: `'start'`, `'center'`, `'end'`, `'spaceBetween'`, `'spaceAround'`, `'spaceEvenly'`

#### `static styles`

`structuralStyles`와 인라인 `css` 블록을 배열로 결합한다.

인라인 블록의 주요 규칙:
- `* { box-sizing: border-box; }`
- `:host { display: flex; flex: var(--weight); }` — 부모 flex 컨테이너에서 `--weight` 변수로 크기 분배
- `section { display: flex; flex-direction: row; width: 100%; min-height: 100%; }` — 내부 수평 레이아웃 컨테이너
- `alignment` 어트리뷰트별 `section`의 `align-items`:
  - `'start'` → `start`
  - `'center'` → `center`
  - `'end'` → `end`
  - `'stretch'` → `stretch`
- `distribution` 어트리뷰트별 `section`의 `justify-content`:
  - `'start'` → `start`
  - `'center'` → `center`
  - `'end'` → `end`
  - `'spaceBetween'` → `space-between`
  - `'spaceAround'` → `space-around`
  - `'spaceEvenly'` → `space-evenly`

#### `render()`

반환 타입: `TemplateResult`

`<section>` 요소를 렌더링한다.
- `class`: `classMap(this.theme.components.Row)`로 테마 클래스를 적용한다.
- `style`: `this.theme.additionalStyles?.Row`가 존재하면 `styleMap()`으로 인라인 스타일을 적용하고, 없으면 `nothing`을 사용한다.
- `<section>` 내부에 `<slot></slot>`을 배치하여 Light DOM 자식을 투영한다.

## 동작 흐름

`Root`로부터 상속받은 `childComponents` 기반 Light DOM 렌더링 메커니즘이 자식 컴포넌트들을 이 엘리먼트의 Light DOM에 삽입한다. `render()`가 생성한 Shadow DOM의 `<slot>`이 자식들을 투영한다. `alignment`와 `distribution` 속성이 어트리뷰트로 반영되어 CSS `:host([...])` 선택자가 `section`의 flex 정렬 스타일을 자동으로 제어한다.
