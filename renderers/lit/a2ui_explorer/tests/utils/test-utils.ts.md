# renderers/lit/a2ui_explorer/tests/utils/test-utils.ts

## 개요

Shadow DOM을 포함한 Lit 기반 컴포넌트 통합 테스트를 지원하는 유틸리티 모음이다. `<local-gallery>` 커스텀 엘리먼트를 마운트하고 특정 예제를 로드하는 헬퍼, Lit의 비동기 업데이트 사이클이 완전히 완료될 때까지 기다리는 헬퍼, 그리고 Shadow DOM 경계를 가로질러 DOM을 탐색·조회하는 유틸리티 함수들을 제공한다. 이 파일에서 내보내는 함수들은 `v0_9/` 하위의 모든 테스트 파일에서 공통으로 사용된다.

## 의존성

### 외부 패키지
- `lit` — `ReactiveElement` 클래스 (Lit 엘리먼트의 업데이트 사이클 감지에 사용)

### 저장소 내부 모듈
- [`../../src/local-gallery`](../../src/local-gallery.ts.md) — `LocalGallery` 커스텀 엘리먼트 클래스

## Exports

| 이름 | 종류 |
|---|---|
| `loadExample` | async 함수 |
| `whenSettled` | async 함수 |
| `walkDeep` | 함수 |
| `getDeepTextContent` | 함수 |
| `querySelectorAllDeep` | 함수 |
| `getSurface` | 함수 |
| `findButtonByText` | async 함수 |

## 상세 명세

### `loadExample(filename: string): Promise<LocalGallery>`

`<local-gallery>` 엘리먼트를 `document.body`에 마운트한 뒤, `filename`에 해당하는 예제를 선택하여 완전히 렌더링될 때까지 기다린 후 엘리먼트를 반환한다.

**동작 단계:**
1. `import('../../src/local-gallery')`를 동적으로 await하여 `<local-gallery>` 커스텀 엘리먼트가 `customElements` 레지스트리에 등록되도록 보장한다.
2. `document.createElement('local-gallery')`로 엘리먼트를 생성하고 `document.body`에 추가한 후 `gallery.updateComplete`를 await한다.
3. `gallery.demoItems.findIndex(item => item.filename === filename)`으로 해당 파일명의 인덱스를 찾는다. 존재하지 않으면 `gallery.remove()`를 호출하여 DOM 오염을 방지하고 `Error: Example not found: ${filename}`을 던진다.
4. `gallery.selectItem(index)`를 호출하여 해당 예제를 활성화한다.
5. `whenSettled(gallery)`를 await하여 자식 컴포넌트까지 완전히 렌더링될 때까지 기다린다.
6. `gallery`를 반환한다.

**매개변수:** `filename: string` — `"02_email-compose.json"` 형태의 A2UI JSON 예제 파일명

**반환값:** `Promise<LocalGallery>`

---

### `whenSettled(root: ParentNode): Promise<void>`

Lit 엘리먼트와 그 자식들의 비동기 업데이트 사이클이 모두 완료될 때까지 재귀적으로 기다린다.

**동작 단계:**
1. `root`가 `ReactiveElement`의 인스턴스이면 `root.updateComplete`를 await한다. 이는 자식 노드들을 안전하게 순회하기 위해 필요하다.
2. `root.children`의 각 직계 자식에 대해 `whenSettled(child)`를 호출하여 `Promise` 배열에 추가한다.
3. `root`가 `Element`이고 `root.shadowRoot`가 존재하면 `whenSettled(root.shadowRoot)`도 배열에 추가한다.
4. `Promise.all(promises)`를 await하여 모든 하위 트리가 완료될 때까지 기다린다.

**매개변수:** `root: ParentNode` — 대기를 시작할 루트 노드

---

### `walkDeep(node: Node, callback: (node: Node) => void): void`

열린(open) Shadow DOM 경계를 가로질러 DOM 트리를 깊이 우선(depth-first) 방식으로 순회한다.

**동작 단계:**
1. 현재 `node`에 대해 `callback(node)`를 즉시 호출한다.
2. `node.childNodes`의 각 자식에 대해 `walkDeep(child, callback)`을 재귀 호출한다.
3. `node`가 `Element`이고 `node.shadowRoot`가 존재하면 `walkDeep(node.shadowRoot, callback)`을 재귀 호출하여 Shadow DOM 안까지 탐색한다.

**매개변수:**
- `node: Node` — 순회를 시작할 노드
- `callback: (node: Node) => void` — 방문하는 모든 노드에 대해 호출되는 함수

---

### `getDeepTextContent(root: Node): string`

Shadow DOM 경계를 가로질러 `root` 하위의 모든 텍스트 노드 내용을 이어 붙인 문자열을 반환한다.

**동작 단계:**
1. 빈 문자열 `text`를 초기화한다.
2. `walkDeep`으로 전체 트리를 순회하면서, 방문한 노드의 `nodeType`이 `Node.TEXT_NODE`인 경우 `node.textContent || ''`를 `text`에 누적한다.
3. 최종 `text`를 반환한다.

**매개변수:** `root: Node`  
**반환값:** `string`

---

### `querySelectorAllDeep(root: Node, selector: string): Element[]`

Shadow DOM 경계를 가로질러 `selector`에 매칭되는 모든 `Element`를 수집하여 배열로 반환한다. 표준 `querySelectorAll`과 동일하게 루트 엘리먼트 자체는 결과에 포함되지 않는다.

**동작 단계:**
1. 빈 배열 `results`를 초기화한다.
2. `walkDeep`으로 전체 트리를 순회하면서, `node !== root`이고 `node instanceof Element`이며 `node.matches(selector)`가 참인 경우에만 `results`에 추가한다.
3. `results`를 반환한다.

**매개변수:**
- `root: Node` — 탐색을 시작할 루트 노드 (결과에 포함되지 않음)
- `selector: string` — CSS 선택자

**반환값:** `Element[]`

---

### `getSurface(root: Element): HTMLElement`

`root` 하위에서 첫 번째 `a2ui-surface` 엘리먼트를 찾아 반환하는 편의 함수다. 해당 엘리먼트가 없으면 `Error: a2ui-surface not found in root element`를 던진다.

**동작 단계:**
1. `querySelectorAllDeep(root, 'a2ui-surface')[0]`으로 첫 번째 일치 엘리먼트를 구한다.
2. 결과가 falsy이면 에러를 던진다.
3. `HTMLElement`로 캐스팅하여 반환한다.

**매개변수:** `root: Element`  
**반환값:** `HTMLElement`

---

### `findButtonByText(surface: HTMLElement, text: string): Promise<HTMLButtonElement>`

Shadow DOM을 포함한 `surface` 하위에서 텍스트 내용에 `text`를 포함하는 첫 번째 `.a2ui-button` 엘리먼트를 찾아 반환한다.

**동작 단계:**
1. `querySelectorAllDeep(surface, '.a2ui-button')`으로 모든 `.a2ui-button` 엘리먼트를 `HTMLButtonElement[]`로 가져온다.
2. `Array.prototype.find`로 `getDeepTextContent(b).includes(text)`가 참인 첫 번째 버튼을 찾는다.
3. 찾지 못하면 `Error: Button with text "${text}" not found`를 던진다.
4. 찾은 버튼을 반환한다.

**매개변수:**
- `surface: HTMLElement` — 탐색 범위가 되는 루트 엘리먼트
- `text: string` — 버튼 텍스트에 포함되어야 하는 문자열

**반환값:** `Promise<HTMLButtonElement>`

## 동작 흐름

파일 로드 시 별도의 초기화 코드는 없다. 각 함수는 독립적으로 호출된다. 일반적인 테스트 흐름은 다음과 같다: `loadExample`으로 `LocalGallery`를 마운트 및 초기화 → `getSurface`로 렌더링된 `a2ui-surface` 엘리먼트 획득 → `getDeepTextContent` / `querySelectorAllDeep` / `findButtonByText`로 콘텐츠 검증 또는 인터랙션 수행 → 인터랙션 후 `whenSettled`로 업데이트 완료 대기.
