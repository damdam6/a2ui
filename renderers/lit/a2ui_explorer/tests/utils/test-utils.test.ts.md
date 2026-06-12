# renderers/lit/a2ui_explorer/tests/utils/test-utils.test.ts

## 개요

`test-utils.ts`에서 내보내는 `walkDeep`, `getDeepTextContent`, `querySelectorAllDeep` 세 함수의 동작을 검증하는 단위 테스트 파일이다. 실제 브라우저 DOM API(Shadow DOM 포함)를 사용하여 각 유틸리티가 Shadow DOM 경계를 올바르게 처리하는지 확인한다. 외부 픽스처나 모킹 없이 `document.createElement`와 `attachShadow`로 직접 DOM 트리를 구성하여 테스트한다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈
- [`./test-utils`](./test-utils.ts.md) — `getDeepTextContent`, `querySelectorAllDeep`, `walkDeep`

## Exports

없음 (테스트 파일)

## 테스트 케이스 명세

### `describe('test-utils')`

#### `describe('walkDeep')`

**`it('should traverse nested shadow DOMs')`**
- 검증 동작: `host1` → Shadow DOM 안의 `inner1` → `inner1` 안의 `host2` → `host2`의 Shadow DOM 안의 `inner2`로 이어지는 2단계 중첩 Shadow DOM 구조를 `walkDeep`이 깊이 우선으로 올바르게 순회한다.
- 픽스처: `container(div)` → `host1(div)` [Shadow: `inner1(div)` → `host2(div)` [Shadow: `inner2(div)`]] 구조를 `document.createElement`와 `attachShadow({mode: 'open'})`으로 직접 구성.
- 기대 결과: `visitedIds`가 `['container', 'host1', 'inner1', 'host2', 'inner2']` 순서로 정확히 일치.

**`it('should preserve document order when interleaving light DOM nodes and shadow DOMs')`**
- 검증 동작: 같은 수준에 Shadow DOM 호스트와 일반 Light DOM 노드가 섞여 있을 때 문서 순서(document order)가 보존된다.
- 픽스처: `container` 아래 `host1`(Shadow: `inner1`), `middle`(일반 div), `host2`(Shadow: `inner2`)를 순서대로 추가.
- 기대 결과: `visitedIds`가 `['container', 'host1', 'inner1', 'middle', 'host2', 'inner2']`.

---

#### `describe('getDeepTextContent')`

**`it('should traverse shadow DOM and return its text')`**
- 검증 동작: Light DOM 텍스트 노드와 Shadow DOM 내부 텍스트(중첩 `<span>` 포함)를 모두 합산하여 하나의 문자열로 반환한다.
- 픽스처: `container`에 직접 추가된 텍스트 노드 `'Light text '`와, Shadow DOM(`innerHTML = 'Shadow text <span>nested shadow text</span>'`)을 갖는 `shadowHost`로 구성.
- 기대 결과: `'Light text Shadow text nested shadow text'`.

---

#### `describe('querySelectorAllDeep')`

**`it('should find elements in the light DOM')`**
- 검증 동작: Shadow DOM 없이 일반 Light DOM 내에 중첩된 `.target` 엘리먼트 두 개를 모두 찾는다.
- 픽스처: `container.innerHTML`로 `id="target1"`, `id="target2"`인 두 개의 `span.target` 생성.
- 기대 결과: `matches.length === 2`, `matches[0].id === 'target1'`, `matches[1].id === 'target2'`.

**`it('should find elements inside shadow DOM')`**
- 검증 동작: Shadow DOM 내부에 있는 `.target` 엘리먼트들도 정확히 찾는다.
- 픽스처: `host`의 Shadow DOM `innerHTML`에 `id="target1"`, `id="target2"`인 두 개의 `span.target` 추가.
- 기대 결과: `matches.length === 2`, `matches.map(m => m.id)`가 `['target1', 'target2']`.

**`it('should not match the root element itself, to match querySelectorAll behavior')`**
- 검증 동작: `container` 자체가 선택자에 매칭되더라도 결과에 포함되지 않는다 — 표준 `querySelectorAll`과 동일한 동작 보장.
- 픽스처: `className = 'target'`인 `container`와, 그 자식으로 역시 `className = 'target'`인 `child`.
- 기대 결과: `matches.length === 1`, `matches[0] === child`.

## 동작 흐름

각 테스트는 `document.createElement`로 픽스처 DOM을 구성하고, 대상 함수를 호출한 뒤 `expect`로 결과를 단언한다. `beforeEach`/`afterEach`는 없으며 각 `it` 블록이 독립적으로 DOM을 구성하고 검증한다.
