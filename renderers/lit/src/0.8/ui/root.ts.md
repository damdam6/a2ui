# renderers/lit/src/0.8/ui/root.ts

## 개요

`Root`는 모든 A2UI Lit 컴포넌트의 기반 클래스이자, `a2ui-root` 커스텀 엘리먼트 자체로서 동작하는 파일이다. `AnyComponentNode` 트리를 수신하여 각 노드 타입에 맞는 Lit 커스텀 엘리먼트 태그로 변환(Light DOM 렌더링)하며, signal 기반 반응형 업데이트를 지원한다. 커스텀 엘리먼트 오버라이드(`componentRegistry` 혹은 `customElements`)를 먼저 탐색하고, 없으면 내장 타입별 `switch` 분기로 폴백한다.

## 의존성

### 외부 패키지
- `@lit-labs/signals` — `SignalWatcher` (signal 구독 지원 믹스인)
- `@lit/context` — `consume` (컨텍스트 소비 데코레이터)
- `lit` — `css`, `html`, `LitElement`, `nothing`, `PropertyValues`, `render`, `TemplateResult`
- `lit/decorators.js` — `customElement`, `property`
- `lit/directives/map.js` — `map`
- `signal-utils/subtle/microtask-effect` — `effect`
- `@a2ui/web_core/data/model-processor` — `A2uiMessageProcessor`
- `@a2ui/web_core/types/primitives` — `StringValue`
- `@a2ui/web_core/types/types` — `AnyComponentNode`, `SurfaceID`, `Theme`

### 저장소 내부 모듈
- [`./context/theme.js`](./context/theme.ts.md) — `themeContext`
- [`./styles.js`](./styles.ts.md) — `structuralStyles`
- [`./component-registry.js`](./component-registry.ts.md) — `componentRegistry`

## Exports

| 이름 | 종류 |
|---|---|
| `Root` | 클래스 (`@customElement('a2ui-root')`, `SignalWatcher(LitElement)` 상속) |

## 상세 명세

### 타입 별칭: `NodeOfType<T>`

시그니처: `type NodeOfType<T extends AnyComponentNode['type']> = Extract<AnyComponentNode, {type: T}>`

`AnyComponentNode` 유니온에서 `type` 필드가 `T`인 멤버만 추출하는 조건부 타입. 파일 내부에서만 사용되며, `switch` 분기의 각 케이스 블록에서 `node` 변수에 구체적인 타입을 부여할 때 활용된다.

---

### 클래스: `Root`

`SignalWatcher(LitElement)`를 상속하며 `@customElement('a2ui-root')`로 등록된다.

#### 필드 및 프로퍼티

| 이름 | 타입 | 기본값 | 데코레이터 | 설명 |
|---|---|---|---|---|
| `surfaceId` | `SurfaceID \| null` | `null` | `@property()` | 이 컴포넌트가 속한 surface 식별자 |
| `component` | `AnyComponentNode \| null` | `null` | `@property()` | 현재 컴포넌트 노드 데이터 |
| `theme` | `Theme` | (컨텍스트 주입) | `@consume({context: themeContext})` | themeContext로 소비되는 테마 |
| `childComponents` | `AnyComponentNode[] \| null` | `null` | `@property({attribute: false})` | 자식 노드 목록 |
| `processor` | `A2uiMessageProcessor \| null` | `null` | `@property({attribute: false})` | 데이터 바인딩 처리기 |
| `dataContextPath` | `string` | `''` | `@property()` | 데이터 컨텍스트 경로 |
| `enableCustomElements` | `boolean` | `false` | `@property()` | 커스텀 엘리먼트 오버라이드 허용 여부 |
| `#weight` | `string \| number` | `1` | — | 내부 weight 저장소 (private class field) |
| `#lightDomEffectDisposer` | `null \| (() => void)` | `null` | — | `effect()` 정리 함수 보관용 private field |

#### 접근자 `weight`

setter(`weight: string | number`): `#weight` 필드에 값을 저장하고, `this.style.setProperty('--weight', \`${weight}\`)`로 CSS 커스텀 프로퍼티 `--weight`를 즉시 반영한다.  
getter: `#weight`를 그대로 반환한다.

#### `static styles`

`structuralStyles` 배열과 인라인 `css` 블록을 결합한 배열. 인라인 블록은 `:host`에 `display: flex`, `flex-direction: column`, `gap: 8px`, `max-height: 80%`를 지정한다.

---

#### `willUpdate(changedProperties: PropertyValues<this>): void`

`childComponents` 프로퍼티 변경이 감지된 경우에만 실행된다.
1. 이미 `#lightDomEffectDisposer`가 존재하면 즉시 호출해 이전 effect 구독을 해제한다.
2. `effect()`를 호출해 새 구독을 등록하고, 반환된 정리 함수를 `#lightDomEffectDisposer`에 저장한다.
3. effect 콜백 내부에서는 `this.childComponents`(signal)를 읽어 구독을 만들고, `renderComponentTree()`로 `TemplateResult`를 생성한 뒤, Lit의 명령형 `render()` 함수로 `this` 자신(Light DOM)에 직접 렌더링한다. `render` 옵션에 `{host: this}`를 전달한다.

#### `disconnectedCallback(): void`

`super.disconnectedCallback()` 호출 후, `#lightDomEffectDisposer`가 존재하면 호출하여 effect 구독을 해제한다.

---

#### `private renderComponentTree(components: AnyComponentNode[] | null): TemplateResult | typeof nothing`

컴포넌트 배열을 Lit `TemplateResult`로 변환하는 핵심 메서드.

- `components`가 `null`이거나 배열이 아니면 `nothing`을 반환한다.
- Lit `map` 디렉티브로 각 `component` 노드를 순회하며 아래 두 경로 중 하나를 선택한다.

**커스텀 엘리먼트 경로** (`enableCustomElements === true`):
1. `componentRegistry.get(component.type)`으로 등록된 생성자를 탐색한다.
2. 없으면 `customElements.get(component.type)`으로 폴백한다.
3. 생성자가 있으면 `new elCtor()`로 인스턴스를 직접 생성한다.
4. `id`, `slot`(slotName이 있을 때), `component`, `weight`(`?? 'initial'`), `processor`, `surfaceId`, `dataContextPath`(`?? '/'`)를 인스턴스에 설정한다.
5. `component.properties`의 모든 엔트리를 `el[prop] = val`로 직접 할당한다(`@ts-expect-error` 사용).
6. `html\`${el}\``을 반환한다.

**내장 타입 `switch` 분기 (폴백)**:  
`component.type` 값에 따라 해당 `<a2ui-*>` 태그를 생성하고 프로퍼티를 바인딩한다. 모든 케이스에서 공통으로 `id`, `slot`(있을 때만), `component`, `weight`(`?? 'initial'`), `processor`, `surfaceId`, `dataContextPath`, `enableCustomElements`를 바인딩한다.

| `type` 값 | 렌더링 태그 | 주요 추가 바인딩 |
|---|---|---|
| `'List'` | `<a2ui-list>` | `direction`(`?? 'vertical'`), `childComponents`(= `node.properties.children`) |
| `'Card'` | `<a2ui-card>` | `childComponents`: `children` 있으면 그대로, 없으면 `child`를 단일 요소 배열로 감쌈 |
| `'Column'` | `<a2ui-column>` | `alignment`(`?? 'stretch'`), `distribution`(`?? 'start'`), `childComponents`(`?? null`) |
| `'Row'` | `<a2ui-row>` | `alignment`(`?? 'stretch'`), `distribution`(`?? 'start'`), `childComponents`(`?? null`) |
| `'Image'` | `<a2ui-image>` | `url`(`?? null`), `usageHint`, `fit` |
| `'Icon'` | `<a2ui-icon>` | `name`(`?? null`) |
| `'AudioPlayer'` | `<a2ui-audioplayer>` | `url`(`?? null`) |
| `'Button'` | `<a2ui-button>` | `action`, `childComponents`(= `[child]`), `primary` |
| `'Text'` | `<a2ui-text>` | `text`, `usageHint`, `model`(= `processor`) — `model`과 `processor` 둘 다 전달 |
| `'CheckBox'` | `<a2ui-checkbox>` | `label`, `value` |
| `'DateTimeInput'` | `<a2ui-datetimeinput>` | `enableDate`(`?? true`), `enableTime`(`?? true`), `value` |
| `'Divider'` | `<a2ui-divider>` | `thickness`, `axis`, `color` |
| `'MultipleChoice'` | `<a2ui-multiplechoice>` | `options`, `maxAllowedSelections`, `selections`, `variant`(`as any`로 접근), `filterable` |
| `'Slider'` | `<a2ui-slider>` | `value`, `minValue`, `maxValue` |
| `'TextField'` | `<a2ui-textfield>` | `label`, `text`, `textFieldType`, `validationRegexp` |
| `'Video'` | `<a2ui-video>` | `url` |
| `'Tabs'` | `<a2ui-tabs>` | `tabItems` 순회 → `titles`(StringValue 배열), `childComponents`(child 배열) 분리 전달 |
| `'Modal'` | `<a2ui-modal>` | `childComponents = [entryPointChild, contentChild]`; 렌더 전 `entryPointChild.slotName = 'entry'`로 돌연변이 |
| `default` | — | `renderCustomComponent(component)` 호출 |

---

#### `private renderCustomComponent(component: AnyComponentNode)`

`switch`의 `default` 케이스를 처리한다.  
- `enableCustomElements`가 `false`이면 아무것도 반환하지 않는다.  
- `componentRegistry` → `customElements.get()`순으로 생성자를 탐색한다.  
- 생성자가 없으면 `` html`Unknown element ${component.type}` ``를 반환한다.  
- 생성자가 있으면 커스텀 엘리먼트 경로와 동일한 방식으로 인스턴스를 생성·설정·반환한다.

#### `render(): TemplateResult | typeof nothing`

Shadow DOM에 `` html`<slot></slot>` ``만 렌더링한다. 실제 컴포넌트 트리는 Light DOM에 직접 렌더링되므로, Shadow DOM의 슬롯이 Light DOM 자식들을 투영한다.

## 동작 흐름

외부에서 `childComponents` 프로퍼티가 설정되면 `willUpdate`가 이전 signal effect를 정리하고 새 effect를 등록한다. effect 내부에서 `childComponents` signal을 구독하여 변경 시 자동으로 `renderComponentTree`를 재실행하고, Lit 명령형 `render()`로 Light DOM을 갱신한다. Shadow DOM에는 `<slot>`만 있으므로 Light DOM에 삽입된 자식들이 슬롯을 통해 노출된다. 컴포넌트가 DOM에서 제거될 때 `disconnectedCallback`이 effect 구독을 해제한다.
