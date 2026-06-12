# renderers/lit/a2ui_explorer/src/local-gallery.ts

## 개요

A2UI 예시들을 브라우저에서 인터랙티브하게 탐색할 수 있는 Lit 커스텀 엘리먼트(`local-gallery`)를 정의한다. 왼쪽 네비게이션 패널에서 예시를 선택하면 `MessageProcessor`를 통해 A2UI 메시지를 단계별로 혹은 한번에 처리하고 결과 surface를 중앙 패널에 렌더링하며, 오른쪽 인스펙터 패널에서 데이터 모델과 액션 로그를 실시간으로 확인할 수 있다. 테마의 primary color를 색상 피커로 실시간 변경할 수 있으며, 자동화된 통합 테스트를 위해 `actionLog` 속성을 공개적으로 노출한다.

## 의존성

### 외부 패키지
- `lit` — `LitElement`, `html`, `nothing`
- `@lit/context` — `provide` 데코레이터
- `lit/decorators.js` — `customElement`, `state` 데코레이터
- `@a2ui/web_core/v0_9` — `MessageProcessor`, `A2uiMessage`, `A2uiClientAction`
- `@a2ui/lit/v0_9` — `basicCatalog`, `Context`
- `@a2ui/markdown-it` — `renderMarkdown`

### 저장소 내부 모듈
- [`./examples`](./examples.ts.md) — `getDemoItems`, `DemoItem`
- [`./local-gallery.css`](./local-gallery.css.ts.md) — `appStyles`

## Exports

- `LocalGallery` (클래스) — `LitElement`를 상속한 `local-gallery` 커스텀 엘리먼트

## 상세 명세

### `LocalGallery` 클래스

`@customElement('local-gallery')`로 등록된 Lit 컴포넌트.

#### 필드 및 상태

| 이름 | 타입 | 데코레이터 | 초기값 | 설명 |
|------|------|-----------|--------|------|
| `mockLogs` | `string[]` | `@state` | `[]` | 인스펙터에 표시할 로그 항목 배열 |
| `demoItems` | `DemoItem[]` | `@state` | `[]` | 로드된 모든 예시 항목 목록 |
| `activeItemIndex` | `number` | `@state` | `0` | 현재 선택된 예시의 인덱스 |
| `processedMessageCount` | `number` | `@state` | `0` | 현재 예시에서 처리된 메시지 수 |
| `currentDataModelText` | `string` | `@state` | `'{}'` | JSON 문자열로 직렬화된 data model 상태 |
| `primaryColor` | `string` | `@state` | `'#1177ee'` | 테마에 적용할 primary color |
| `actionLog` | `A2uiClientAction[]` | 없음 | `[]` | 디스패치된 액션 목록 (통합 테스트용 공개 속성) |
| `markdownRenderer` | — | `@provide({ context: Context.markdown })` `private @state` | `renderMarkdown` | Lit context를 통해 하위 컴포넌트에 제공되는 마크다운 렌더러 |
| `processor` | `MessageProcessor` | `private` | 새 인스턴스 | `[basicCatalog]`와 액션 콜백으로 초기화된 메시지 프로세서 |
| `dataModelSubscription` | `{unsubscribe: () => void} \| undefined` | `private` | `undefined` | data model 구독 핸들 (정리 시 사용) |

`static styles = [appStyles]`

#### `connectedCallback(): Promise<void>`

컴포넌트가 DOM에 연결될 때 호출된다. `super.connectedCallback()`을 먼저 호출한 뒤, `processor.model.onSurfaceCreated`를 구독하여 새 surface가 생성될 때마다 해당 surface의 `onError`를 구독하고 오류를 `log()`로 기록한다. 이후 `loadExamples()`를 호출한다.

#### `loadExamples(): void`

`getDemoItems()`를 호출하여 `demoItems`를 채운다. 항목이 하나 이상 있으면 `selectItem(0)`으로 첫 번째 항목을 선택한다. 오류 발생 시 `console.error`로 기록한다.

#### `selectItem(index: number): void`

예시 항목을 전환한다. 먼저 `deleteActiveExampleSurface()`로 이전 예시의 surface를 삭제한 뒤 `activeItemIndex`를 `index`로 갱신하고 `reloadExample()`을 호출한다.

#### `resetSurface(): void`

현재 surface의 모든 상태를 초기화한다:
1. `processedMessageCount`, `mockLogs`, `currentDataModelText`, `actionLog`를 초기값으로 재설정한다.
2. `dataModelSubscription`이 있으면 `unsubscribe()`를 호출하고 `undefined`로 설정한다.
3. `deleteActiveExampleSurface()`를 호출하여 surface를 삭제한다.

#### `deleteActiveExampleSurface(): void`

`activeItemIndex`에 해당하는 `surfaceId`를 찾아 processor 모델에 해당 surface가 존재하면 `deleteSurface` 메시지(`{version: 'v0.9', deleteSurface: {surfaceId}}`)를 처리한다.

#### `advanceMessages(all: boolean = false): void`

메시지 처리를 진행한다.
1. 활성 항목이 없으면 즉시 반환한다.
2. `all`이 `true`이면 `processedMessageCount`부터 끝까지의 나머지 메시지 전체를, `false`이면 `processedMessageCount` 인덱스의 메시지 하나만 처리 대상으로 선택한다.
3. 처리 대상이 없으면 반환한다.
4. `applyPrimaryColorToMessages()`로 색상을 주입한 뒤 `processor.processMessages()`에 전달한다.
5. `processedMessageCount`를 처리된 메시지 수만큼 증가시킨다.
6. `dataModelSubscription`이 없으면 surface의 `dataModel.subscribe('/', ...)`를 통해 data model 변경 시 `currentDataModelText`를 업데이트하는 구독을 설정한다.

#### `reloadExample(): void` (private)

`resetSurface()`로 초기화한 뒤 `advanceMessages(true)`로 전체 메시지를 한번에 처리한다. 예시 전환 또는 테마 변경 시 사용된다.

#### `applyPrimaryColorToMessages(messages: A2uiMessage[]): A2uiMessage[]` (private)

메시지 배열을 순회하여 `createSurface`를 포함하는 메시지에 한해 `theme.primaryColor`를 `this.primaryColor` 값으로 주입한 새 메시지 객체를 생성한다(불변 변환). 다른 타입의 메시지는 그대로 반환한다. 새 배열을 반환한다.

#### `onColorInput(e: Event): void` (private)

`input[type=color]`의 `input` 이벤트 핸들러. `e.target`에서 `HTMLInputElement`로 캐스트하여 `value`를 `primaryColor`에 할당하고 `reloadExample()`을 호출한다.

#### `clearColor(): void` (private)

`primaryColor`를 빈 문자열로 설정하고 `reloadExample()`을 호출한다.

#### `log(msg: string, detail?: any): void`

현재 시각(`toLocaleTimeString()`)을 포함한 로그 항목을 생성한다. `detail`이 있으면 `JSON.stringify(detail, null, 2)`를 개행으로 이어 붙인다. 새 항목을 기존 `mockLogs` 배열 뒤에 추가한 새 배열을 `mockLogs`에 할당한다(불변 갱신).

#### `render(): TemplateResult`

Lit 템플릿을 반환한다. 구조:

- `<header>` — "A2UI Explorer" 제목 및 "v0.9 Basic Catalog" 부제
- `<main>`:
  - `<nav class="nav-pane">` — `demoItems`를 순회하여 각 항목에 대한 `.nav-item` 렌더링. 활성 항목에 `active` 클래스 적용. 클릭 시 `selectItem(i)` 호출.
  - `<section class="gallery-pane">`:
    - `.preview-header` — 제목, 설명, 메시지 제어 버튼(Reset / +1 Message / All Messages), 색상 피커
    - `.preview-content` — surface가 있으면 `<a2ui-surface .surface=${surface}>` 렌더링, 없으면 안내 텍스트
  - `<aside class="inspector-pane">`:
    - Data Model 섹션 — `currentDataModelText` 표시
    - Action Logs 섹션 — `mockLogs`가 비어 있으면 "No actions logged..." 표시, 있으면 각 로그를 `.log-entry`로 렌더링. `column-reverse` 방향이므로 최신 로그가 위에 표시됨.

## 동작 흐름

컴포넌트가 DOM에 연결되면(`connectedCallback`) surface 오류 구독을 설정하고 예시를 로드한다. 사용자가 네비게이션 항목을 클릭하면 이전 surface가 삭제되고 새 예시가 전체 메시지와 함께 즉시 로드된다. 메시지 제어 버튼으로 메시지를 단계별 또는 한번에 처리할 수 있으며, 처리 시마다 data model 구독이 설정되어 인스펙터 패널이 실시간으로 갱신된다. 색상 피커 변경 시 예시가 새 primary color와 함께 재로드된다.
