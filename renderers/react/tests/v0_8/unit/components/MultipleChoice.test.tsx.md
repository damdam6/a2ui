# renderers/react/tests/v0_8/unit/components/MultipleChoice.test.tsx

## 개요

A2UI 명세에 따라 `MultipleChoice` 컴포넌트의 렌더링·DOM 구조·사용자 상호작용을 검증하는 단위 테스트 파일이다. `MultipleChoice`는 `selections`(path 참조 객체)와 `options`(label·value 쌍 배열)를 필수 프로퍼티로 받으며, Lit 렌더러 동작과 일치하는 `<select>` 드롭다운으로 렌더링되어야 한다. `maxAllowedSelections`은 선택 프로퍼티다.

## 의존성

### 외부 패키지
- `vitest` — `describe`, `it`, `expect`
- `@testing-library/react` — `render`, `fireEvent`
- `react` — `React`

### 저장소 내부 모듈
- [`../../utils`](../../utils/index.ts.md) — `TestWrapper`, `TestRenderer`, `createSimpleMessages`

## Exports

이 파일은 아무것도 export하지 않는다.

## 테스트 케이스 명세

모든 테스트는 `createSimpleMessages(id, 'MultipleChoice', { selections, options })` 헬퍼로 메시지를 생성한다. `selections` 필드는 항상 `{path: '/mcSelections'}` 형태의 경로 참조 객체이며, `options`는 `{label: {literalString: string}, value: string}` 배열이다.

### describe: `MultipleChoice Component`

#### describe: `Basic Rendering`

| 테스트명 | 검증 동작 | 픽스처/모킹 |
|---|---|---|
| `should render a select element` | `select` 요소가 DOM에 존재하고 `tagName`이 `'SELECT'`이다 | 옵션 2개(`'Option A'`/`'a'`, `'Option B'`/`'b'`) |
| `should render with wrapper div having correct class` | `.a2ui-multiplechoice` 요소가 DOM에 존재하고 `tagName`이 `'DIV'`이다 | 옵션 1개 |
| `should render all option labels` | `option` 요소가 3개이며 `textContent`가 순서대로 `'First Option'`, `'Second Option'`, `'Third Option'`이다 | 옵션 3개 |
| `should render correct number of options` | `option` 요소가 정확히 3개이다 | 옵션 3개 (`'A'`/`'a'`, `'B'`/`'b'`, `'C'`/`'c'`) |
| `should render different options for different inputs` | 인스턴스1은 `'Alpha'`만 포함하고 `'Beta'`는 없으며, 인스턴스2는 `'Beta'`·`'Gamma'`를 포함하고 `'Alpha'`는 없다 | 두 개의 독립 `render` 호출 |

#### describe: `Description Label`

| 테스트명 | 검증 동작 | 픽스처/모킹 |
|---|---|---|
| `should render default description when not provided` | `label` 요소가 존재하고 `textContent`가 `'Select an item'`이다 | `description` 필드 미지정 |
| `should associate label with select via htmlFor` | `label`의 `for` 속성(`getAttribute('for')`)이 `select`의 `id` 값과 동일하다 | 기본 메시지 |

#### describe: `Option Values`

| 테스트명 | 검증 동작 | 픽스처/모킹 |
|---|---|---|
| `should set correct value attributes on options` | 3개 `option`의 `value`가 순서대로 `'sm'`, `'md'`, `'lg'`이다 | `options: [{value:'sm'}, {value:'md'}, {value:'lg'}]` |

#### describe: `User Interaction`

| 테스트명 | 검증 동작 | 픽스처/모킹 |
|---|---|---|
| `should update select value on change` | `fireEvent.change(select, {target: {value: 'b'}})` 후 `select.value`가 `'b'`이다 | 옵션 `'a'`, `'b'`, `'c'` |
| `should handle multiple sequential changes` | `'a'` → `'c'` → `'b'` 순서로 `fireEvent.change`를 세 번 발생시켜, 각 단계 직후 `select.value`가 기대값과 일치한다 | 동일 `select` 요소에 연속 이벤트 |

#### describe: `Structure`

| 테스트명 | 검증 동작 | 픽스처/모킹 |
|---|---|---|
| `should have correct DOM structure: div > section > label + select` | `.a2ui-multiplechoice(DIV)` 안의 `section`이 정확히 2개 자식을 가지며, 첫 번째가 `LABEL`, 두 번째가 `SELECT`이다 | 옵션 2개 |
| `should have select inside section container` | `section` 안에 `select`가 존재한다 | 옵션 1개 |
| `should have options inside select` | `select` 안에 `option`이 2개이다 | 옵션 2개 |

## 동작 흐름

1. 각 테스트에서 `createSimpleMessages('mc-*', 'MultipleChoice', { selections, options })`로 `ServerToClientMessage[]`를 생성한다.
2. `<TestWrapper><TestRenderer messages={...} /></TestWrapper>`로 DOM에 마운트한다.
3. `container.querySelector` / `querySelectorAll`로 대상 요소를 선택하고, `expect` 단언으로 존재 여부·태그명·속성 값·텍스트 내용을 검증한다.
4. 사용자 상호작용 테스트에서는 `fireEvent.change`로 `select` 값 변경을 시뮬레이션하고 `select.value`를 직접 단언한다.
5. 비교 테스트(`different inputs`)는 두 개의 독립 컨테이너를 각각 렌더링하여 텍스트 포함 여부를 교차 검증한다.
