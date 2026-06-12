# renderers/react/tests/v0_8/unit/components/List.test.tsx

## 개요

A2UI 명세에 따라 `List` 컴포넌트의 렌더링 동작을 검증하는 단위 테스트 파일이다. `createListMessages` 헬퍼로 `List` 컴포넌트 메시지를 조립하고, `TestWrapper` / `TestRenderer`를 통해 React DOM에 마운트한 뒤 클래스명·속성·DOM 구조가 스펙과 일치하는지 확인한다. `List`의 필수 프로퍼티는 `children`, 선택 프로퍼티는 `direction`(`'vertical'` | `'horizontal'`)과 `alignment`(`'start'` | `'center'` | `'end'` | `'stretch'`)이다.

## 의존성

### 외부 패키지
- `vitest` — `describe`, `it`, `expect`
- `@testing-library/react` — `render`
- `react` — `React`
- `@a2ui/web_core/types/types` — 타입 전용 import (`Types.ServerToClientMessage`)

### 저장소 내부 모듈
- [`../../utils`](../../utils/index.ts.md) — `TestWrapper`, `TestRenderer`, `createSurfaceUpdate`, `createBeginRendering`

## Exports

이 파일은 아무것도 export하지 않는다. 모든 심볼은 테스트 런타임 내부에서만 사용된다.

## 테스트 케이스 명세

### 헬퍼 함수: `createListMessages`

**시그니처:**
```
function createListMessages(
  id: string,
  props: {
    childIds: string[];
    childTexts?: string[];
    direction?: 'vertical' | 'horizontal';
    alignment?: 'start' | 'center' | 'end' | 'stretch';
  },
  surfaceId = '@default',
): Types.ServerToClientMessage[]
```

**동작 로직:**
1. `props.childIds`를 순회하여 각 항목에 대해 `Text` 컴포넌트 객체를 생성한다. 텍스트 값은 `props.childTexts?.[index]`가 있으면 그 값을, 없으면 `` `Item ${index + 1}` `` 형태의 기본값을 사용한다.
2. `createSurfaceUpdate` 호출 시 생성된 `Text` 노드 배열 뒤에 `List` 컴포넌트 객체를 추가한다. `List`의 `children` 필드는 `{explicitList: props.childIds}` 형태이며, `direction`과 `alignment`는 제공된 경우에만 포함된다.
3. `createBeginRendering(id, surfaceId)`를 두 번째 메시지로 추가하여 2-요소 배열을 반환한다.

---

### describe: `List Component`

#### describe: `Basic Rendering`

| 테스트명 | 검증 동작 | 픽스처/모킹 |
|---|---|---|
| `should render a list container` | `.a2ui-list` 요소가 DOM에 존재한다 | `childIds: ['item-1', 'item-2']` |
| `should render section element inside list` | `.a2ui-list` 안에 `section` 요소가 존재한다 | `childIds: ['item-1', 'item-2']` |
| `should render all children` | 3개 자식의 텍스트(`'First'`, `'Second'`, `'Third'`)가 모두 `container.textContent`에 포함된다 | `childIds` 3개, `childTexts: ['First', 'Second', 'Third']` |
| `should render correct number of children` | `.a2ui-text` 요소가 정확히 4개 존재한다 | `childIds` 4개 |
| `should render different number of items for different inputs` | 2-자식 렌더에서 `.a2ui-text` 2개, 4-자식 렌더에서 4개가 나오며 두 값이 다르다 | 두 개의 독립 `render` 호출 |

#### describe: `Direction`

| 테스트명 | 검증 동작 | 픽스처/모킹 |
|---|---|---|
| `should default to vertical direction` | `direction` 미지정 시 `.a2ui-list`의 `data-direction` 속성이 `'vertical'`이다 | `direction` 생략 |
| `should set horizontal direction when specified` | `direction: 'horizontal'` 지정 시 `data-direction`이 `'horizontal'`이고 `'vertical'`이 아니다 | `direction: 'horizontal'` |
| `should set vertical direction when specified` | `direction: 'vertical'` 명시 지정 시 `data-direction`이 `'vertical'`이다 | `direction: 'vertical'` |
| `should render different directions for different inputs` | 수평/수직 두 인스턴스의 `data-direction`이 각각 `'horizontal'`·`'vertical'`이며 서로 다르다 | 두 개의 독립 `render` 호출 |

#### describe: `Empty List`

| 테스트명 | 검증 동작 | 픽스처/모킹 |
|---|---|---|
| `should render empty list container` | `children: {explicitList: []}` 로 만든 `List`에서도 `.a2ui-list`와 내부 `section`이 DOM에 존재한다 | 헬퍼 없이 직접 `createSurfaceUpdate` + `createBeginRendering` 조립 |

#### describe: `Structure`

| 테스트명 | 검증 동작 | 픽스처/모킹 |
|---|---|---|
| `should have correct DOM structure: div > section > children` | `.a2ui-list` 루트가 `DIV` 태그이고, 그 안에 `section`이 있으며, `section` 내부에 `.a2ui-text` 요소가 2개 존재한다 | `childIds: ['item-1', 'item-2']` |

## 동작 흐름

1. 각 테스트에서 `createListMessages`(또는 직접 조립한 메시지 배열)로 `ServerToClientMessage[]`를 생성한다.
2. `<TestWrapper><TestRenderer messages={...} /></TestWrapper>`로 렌더링한다.
3. `container.querySelector` / `querySelectorAll`로 대상 요소를 선택하고 `expect` 단언으로 존재 여부·속성 값·요소 수·텍스트 내용을 검증한다.
4. 방향 테스트는 `data-direction` HTML 속성 값을 읽어 판단한다.
5. 구조 테스트는 `DIV > section > .a2ui-text` 계층을 순서대로 확인한다.
6. 비교 테스트(`different inputs`)는 두 개의 독립 컨테이너를 각각 렌더링해 결과를 대조한다.
