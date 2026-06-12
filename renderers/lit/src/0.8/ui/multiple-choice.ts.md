# renderers/lit/src/0.8/ui/multiple-choice.ts

## 개요

`<a2ui-multiplechoice>` Web Component를 정의한다. 체크박스 드롭다운(`checkbox` variant)과 칩(`chips` variant) 두 가지 레이아웃으로 다중 선택 UI를 제공한다. 선택 상태는 `Primitives.StringValue`(바인딩 경로) 또는 `string[]`(직접 배열)로 관리하며, `filterable` 속성으로 옵션 필터링 입력을 활성화할 수 있다.

## 의존성

### 외부 패키지
- `lit` — `html`, `css`, `nothing`
- `lit/decorators.js` — `customElement`, `property`, `state`

### 저장소 내부 모듈
- [`./root.js`](../root.ts.md) — 베이스 클래스 `Root`
- [`./styles.js`](../styles.ts.md) — `structuralStyles`
- [`./utils/utils.js`](utils/utils.ts.md) — `extractStringValue`
- `@a2ui/web_core/data/model-processor` — `A2uiMessageProcessor` (외부 패키지)
- `@a2ui/web_core/types/primitives` — `Primitives` 네임스페이스 (외부 패키지)

## Exports

| 이름 | 종류 | 설명 |
|---|---|---|
| `MultipleChoice` | 클래스 (LitElement 서브클래스) | `<a2ui-multiplechoice>` 커스텀 엘리먼트 |

## 상세 명세

### `@customElement('a2ui-multiplechoice') class MultipleChoice extends Root`

**Properties (Lit @property)**

- `accessor description: string | null = null` — 헤더 텍스트 또는 안내 문구. chips variant에서는 소제목으로, checkbox variant에서는 미선택 시 드롭다운 헤더 텍스트로 사용된다.
- `accessor options: { label: Primitives.StringValue; value: string }[] = []` — 선택 가능한 옵션 목록. `label`은 표시용 `StringValue`, `value`는 선택 값 식별자.
- `accessor selections: Primitives.StringValue | string[] = []` — 현재 선택 상태. 배열이면 직접 관리, `StringValue`(path 포함 객체)이면 `processor`를 통해 읽고 씀.
- `accessor variant: 'checkbox' | 'chips' = 'checkbox'` — UI 레이아웃 방식.
- `accessor filterable: boolean = false` — `{ type: Boolean }`. `true`이면 필터 입력창 표시.

**States (Lit @state)**

- `accessor isOpen: boolean = false` — checkbox variant의 드롭다운 열림 여부.
- `accessor filterText: string = ''` — 필터 입력 텍스트.

**Static 필드**

- `static styles` — `structuralStyles`와 로컬 CSS 배열. Material Design 3 토큰(`--md-sys-color-*`, `--md-sys-elevation-*`) 기반의 드롭다운, 칩, 체크박스 스타일을 포함한다.
  - `.container`: flex column, gap 4px.
  - `.dropdown-header`: flex, surface 배경, outline-variant 테두리, 8px 둥근 모서리, 클릭 커서, elevation-level1 그림자.
  - `.dropdown-wrapper`: 닫힘 시 `display: none`, `.open`일 때 `display: flex`, 최대 높이 300px, overflow hidden.
  - `.filter-container`: padding 8px, outline-variant 하단 테두리, flex-shrink 0.
  - `.option-item`: flex, gap 12px, padding 12px 16px, `.selected`이면 체크박스 색 변경.
  - `.chip`: inline-flex, border-radius 16px, `.selected`이면 secondary-container 배경/색.
  - `.chip-icon`: 기본 `display: none`, `.selected`이면 `display: block`.
  - `@keyframes fadeIn`: opacity 0→1, translateY -8px→0.

---

#### `#setBoundValue(value: string[]): void`

Private 메서드. `selections`이 path 바인딩 객체일 때 새 선택값 배열을 `processor.setData`로 저장한다.
1. `this.selections`이 없거나 `this.processor`가 없으면 반환.
2. `'path' in this.selections`가 아니면 반환.
3. `this.selections.path`가 falsy이면 반환.
4. `this.processor.setData(this.component, this.selections.path, value, surfaceId)`를 호출한다. `surfaceId`는 `this.surfaceId ?? A2uiMessageProcessor.DEFAULT_SURFACE_ID`.

---

#### `getCurrentSelections(): string[]`

Public 메서드. 현재 선택된 값의 배열을 반환한다.
1. `Array.isArray(this.selections)`이면 그대로 반환.
2. `this.processor` 또는 `this.component`가 없으면 빈 배열 반환.
3. `this.processor.getData(this.component, this.selections.path!, surfaceId)`로 값 조회.
4. 결과가 배열이면 `string[]`로 캐스팅해 반환, 그렇지 않으면 빈 배열 반환.

---

#### `toggleSelection(value: string): void`

Public 메서드. 특정 값의 선택 상태를 토글한다.
1. `getCurrentSelections()`로 현재 배열을 가져온다.
2. `current.includes(value)`이면 해당 값을 제거한 새 배열로 `#setBoundValue`를 호출.
3. 그렇지 않으면 해당 값을 추가한 새 배열로 `#setBoundValue`를 호출.
4. `this.requestUpdate()`를 호출해 강제 재렌더링을 유도한다. (selections가 배열 참조일 때 Lit이 변경을 감지하지 못하는 경우를 대비)

---

#### `#renderCheckIcon(): TemplateResult`

Private 메서드. 체크 표시 SVG를 반환한다. SVG path는 `"M382-240 154-468l57-57 171 171 367-367 57 57-424 424Z"` (Material Symbols done 아이콘). 클래스 `chip-icon`을 가지며 `.selected .chip-icon`에서 `display: block`이 된다.

---

#### `#renderFilter(): TemplateResult`

Private 메서드. 옵션 필터링을 위한 텍스트 입력창을 반환한다.
- `type="text"`, 클래스 `filter-input`, placeholder `"Filter options..."`.
- `.value`는 `this.filterText`로 바인딩.
- `@input` 핸들러: `e.target.value`를 `this.filterText`에 대입.
- `@click` 핸들러: `e.stopPropagation()`으로 드롭다운 토글 이벤트 전파를 막는다.

---

#### `render(): TemplateResult`

1. `getCurrentSelections()`로 현재 선택값을 가져온다.
2. `this.filterText`를 기준으로 `this.options`를 필터링한다. 각 옵션의 label을 `extractStringValue(option.label, this.component, this.processor, this.surfaceId)`로 해석해 `filterText`를 대소문자 무시하여 포함하는지 확인한다.

**chips variant (`this.variant === 'chips'`):**
- `.container` div를 반환한다.
- `description`이 있으면 `<div class="header-text">`로 표시.
- `filterable`이 true이면 `#renderFilter()` 삽입.
- `.chips-container` 안에 각 `filteredOptions`를 chip으로 렌더링. 선택된 경우 `class="chip selected"`, `#renderCheckIcon()` 표시, `@click`에서 `toggleSelection(option.value)` 호출 및 `e.stopPropagation()`.
- 옵션이 없으면 `"No options found"` 텍스트 표시.

**checkbox variant (기본):**
- `currentSelections.length > 0`이면 헤더 텍스트는 `"${count} Selected"`, 아니면 `this.description ?? 'Select items'`.
- `.dropdown-header` 클릭 시 `this.isOpen = !this.isOpen` 토글.
- `isOpen`에 따라 `.chevron` 요소의 클래스에 `open` 추가 여부 결정. chevron SVG는 아래 화살표 아이콘.
- `.dropdown-wrapper`에 `isOpen ? 'open' : ''` 클래스 적용. `open` 클래스일 때 `display: flex`로 전환.
- `filterable`이면 `#renderFilter()` 표시.
- `.options-scroll-container` 안에 각 `filteredOptions`를 `.option-item`으로 렌더링. 선택된 경우 `class="option-item selected"`. 체크박스 표시는 `.checkbox` div 내 `.checkbox-icon` span(`"✓"` 문자). `@click`에서 `toggleSelection` 및 `stopPropagation`.
- 옵션 없으면 `"No options found"` 텍스트 표시.

## 동작 흐름

컴포넌트는 `variant`에 따라 chips 또는 checkbox 드롭다운 두 가지 렌더링 경로를 가진다. 사용자 클릭 시 `toggleSelection`이 호출되어 `#setBoundValue`로 `processor`에 새 선택 배열을 쓰고 `requestUpdate`로 재렌더링을 유발한다. `filterText` 상태가 변경되면 Lit이 자동으로 재렌더하며 `filteredOptions`가 재계산된다. `selections`이 배열이면 컴포넌트 자체가 상태를 소유하고, path 바인딩이면 외부 `processor` 데이터 스토어가 상태를 소유한다.
