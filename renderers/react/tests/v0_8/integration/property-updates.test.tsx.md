# renderers/react/tests/v0_8/integration/property-updates.test.tsx

## 개요

서버 주도(server-driven) 방식의 `surfaceUpdate` 메시지를 통해 A2UI 컴포넌트의 속성이 올바르게 갱신되는지 검증하는 통합 테스트 파일이다. "push 모델", 즉 사용자 인터랙션이 아닌 서버가 새 속성을 전송할 때 렌더링이 반영되는 흐름에만 집중한다. Content, Interactive, Layout, Complex의 네 가지 컴포넌트 카테고리에 걸쳐 총 13개 테스트 케이스를 포함하며, 각 케이스는 초기 렌더링 이후 10ms 타이머를 이용해 두 번째 `surfaceUpdate`를 전송하고 `waitFor`로 DOM 변화를 비동기 검증한다.

## 의존성

### 외부 패키지
- `vitest` — `describe`, `it`, `expect`
- `@testing-library/react` — `render`, `screen`, `waitFor`
- `react` — `React`, `useEffect`

### 저장소 내부 모듈
- [`../../../src/v0_8`](../../../src/v0_8.md) — `A2UIProvider`, `A2UIRenderer`, `useA2UI` (v0_8 공개 API)
- [`../utils`](../utils/index.ts.md) — `createSurfaceUpdate`, `createBeginRendering` (테스트 메시지 팩토리)

## Exports

없음 (테스트 파일, export 없음).

## 테스트 케이스 명세

각 테스트는 동일한 2단계 패턴을 따른다.
1. `useEffect` 안에서 `stage === 'initial'` 일 때 `processMessages`로 초기 컴포넌트를 등록하고 `createBeginRendering`으로 렌더링을 시작한다. `setTimeout(..., 10)`으로 10ms 후 `stage`를 `'updated'`로 전환한다.
2. `stage === 'updated'`가 되면 변경된 속성을 담은 `surfaceUpdate`만 다시 전송한다(`createBeginRendering` 없음).
3. 초기 단계 직후 즉시 초기 상태를 동기 검증한 뒤, `waitFor`로 비동기적으로 `'updated'` 상태와 갱신된 DOM을 검증한다.

렌더러 컴포넌트마다 `<span data-testid="stage">`를 두어 현재 단계를 DOM에 표시한다.

---

### Content Components

#### `should update Text content`
- **검증 동작**: `Text` 컴포넌트의 `text.literalString`이 `'Original text'`에서 `'Updated text'`로 변경된다. 갱신 후 원본 텍스트는 DOM에서 사라진다.
- **픽스처/모킹**: 컴포넌트 id `'text-1'`, `usageHint: 'body'`. 두 단계 모두 동일 id를 사용하며 텍스트 값만 다르다.

#### `should update Text usageHint`
- **검증 동작**: `Text` 컴포넌트의 `usageHint`가 `'h1'`에서 `'caption'`으로 변경될 때 텍스트 `'Heading'`이 DOM에 유지됨을 확인한다. `usageHint`는 HTML 태그가 아닌 CSS 클래스에 영향을 주므로, 태그 변경이 아니라 요소 존재 여부만 검사한다.
- **픽스처/모킹**: 컴포넌트 id `'text-1'`, 텍스트 값은 두 단계 모두 `'Heading'`으로 동일.

#### `should update Image url`
- **검증 동작**: `Image` 컴포넌트의 `url.literalString`이 `'https://example.com/old.jpg'`에서 `'https://example.com/new.jpg'`로 변경된다. `container.querySelector('img')`의 `src` 속성으로 검증.
- **픽스처/모킹**: 컴포넌트 id `'img-1'`, `usageHint: 'mediumFeature'` 유지.

#### `should update Image usageHint`
- **검증 동작**: `Image` 컴포넌트의 `usageHint`가 `'icon'`에서 `'avatar'`로 변경될 때 `.a2ui-image img` 요소가 DOM에 유지됨을 확인. `usageHint`는 CSS 클래스/테마에 영향을 주므로 data 속성이 아닌 요소 존재 여부로만 판단한다.
- **픽스처/모킹**: 컴포넌트 id `'img-1'`, url은 `'https://example.com/img.jpg'`로 동일.

#### `should update Icon name`
- **검증 동작**: `Icon` 컴포넌트의 `name.literalString`이 `'home'`에서 `'settings'`로 변경된다. `.g-icon` 요소의 text content로 검증.
- **픽스처/모킹**: 컴포넌트 id `'icon-1'`.

---

### Interactive Components

#### `should update Button child text`
- **검증 동작**: `Button`의 자식 `Text` 컴포넌트(`'btn-text'`)의 텍스트가 `'Click me'`에서 `'Submit now'`로 변경된다. `getByRole('button', {name: ...})`으로 접근성 이름을 검증하며, 이전 텍스트 버튼은 사라진다.
- **픽스처/모킹**: 텍스트 컴포넌트 id `'btn-text'`, 버튼 id `'btn-1'`, action `{name: 'submit'}` 유지.

#### `should update TextField label`
- **검증 동작**: `TextField`의 `label.literalString`이 `'Username'`에서 `'Email address'`로 변경된다. 이전 라벨 텍스트는 DOM에서 사라진다.
- **픽스처/모킹**: 컴포넌트 id `'field-1'`.

#### `should update CheckBox label`
- **검증 동작**: `CheckBox`의 `label.literalString`이 `'Accept terms'`에서 `'I agree to the terms and conditions'`로 변경된다. 이전 라벨은 DOM에서 사라진다.
- **픽스처/모킹**: 컴포넌트 id `'cb-1'`, `value: {literalBoolean: false}` 두 단계 모두 유지.

#### `should update CheckBox checked state`
- **검증 동작**: `CheckBox`의 `value.literalBoolean`이 `false`에서 `true`로 변경된다. `getByRole('checkbox')`의 체크 여부로 검증.
- **픽스처/모킹**: 컴포넌트 id `'cb-1'`, 라벨 `'Option'` 두 단계 모두 유지.

#### `should update Slider value`
- **검증 동작**: `Slider`의 `value.literalNumber`가 `25`에서 `75`로 변경된다. `getByRole('slider')`의 value 속성(문자열 `'25'` → `'75'`)으로 검증.
- **픽스처/모킹**: 컴포넌트 id `'slider-1'`, `minValue: 0`, `maxValue: 100` 유지.

#### `should update Slider min and max values`
- **검증 동작**: `Slider`의 `minValue`가 `0`에서 `10`으로, `maxValue`가 `100`에서 `200`으로 변경된다. `min`/`max` HTML 속성으로 검증. `value`는 `50`으로 고정.
- **픽스처/모킹**: 컴포넌트 id `'slider-1'`.

---

### Layout Components

#### `should update Column alignment`
- **검증 동작**: `Column`의 `alignment`가 `'start'`에서 `'center'`로 변경된다. `.a2ui-column` 요소의 `data-alignment` 속성으로 검증.
- **픽스처/모킹**: 컴포넌트 id `'col-1'`, 자식 텍스트 id `'text-1'` (값 `'Content'`), `children: {explicitList: ['text-1']}` 유지.

#### `should update Column distribution`
- **검증 동작**: `Column`의 `distribution`이 `'start'`에서 `'spaceBetween'`으로 변경된다. `.a2ui-column` 요소의 `data-distribution` 속성으로 검증.
- **픽스처/모킹**: 컴포넌트 id `'col-1'`, 자식 텍스트 id `'text-1'` 유지.

#### `should update Row alignment`
- **검증 동작**: `Row`의 `alignment`가 `'start'`에서 `'end'`로 변경된다. `.a2ui-row` 요소의 `data-alignment` 속성으로 검증.
- **픽스처/모킹**: 컴포넌트 id `'row-1'`, 자식 텍스트 id `'text-1'` 유지.

#### `should update List direction`
- **검증 동작**: `List`의 `direction`이 `'vertical'`에서 `'horizontal'`로 변경된다. `.a2ui-list` 요소의 `data-direction` 속성으로 검증.
- **픽스처/모킹**: 컴포넌트 id `'list-1'`, 자식 텍스트 id `'item-1'` (값 `'Item'`) 유지.

---

### Complex Components

#### `should update Tabs titles`
- **검증 동작**: `Tabs`의 `tabItems[0].title.literalString`이 `'Tab A'`에서 `'Renamed Tab'`으로 변경된다. `getByRole('button', {name: ...})`으로 검증하며, 이전 탭 버튼은 사라진다.
- **픽스처/모킹**: 컴포넌트 id `'tabs-1'`, 콘텐츠 id `'content-1'` (텍스트 `'Tab content'`) 유지.

#### `should add new tabs via surfaceUpdate`
- **검증 동작**: 초기에 1개였던 탭(`'Tab 1'`)에 새 탭(`'Tab 2'`)이 추가된다. 업데이트 후 두 탭 모두 DOM에 존재한다.
- **픽스처/모킹**: 초기에는 `content-1`만 등록, 업데이트 시 `content-2` (텍스트 `'Content 2'`)를 추가로 등록하고 `tabItems`를 2개로 확장.

## 동작 흐름

1. 각 테스트는 `A2UIProvider`로 감싼 로컬 렌더러 컴포넌트를 `render`한다.
2. 렌더러 컴포넌트는 `useA2UI()`에서 `processMessages`를 꺼내고 `React.useState`로 `'initial' | 'updated'` 단계를 관리한다.
3. `useEffect`가 `stage` 변화에 반응하여 해당 단계의 메시지를 `processMessages`에 전달한다. 초기 단계에서는 `createSurfaceUpdate` + `createBeginRendering`을 전송하고 10ms 후 단계를 전환하며, 업데이트 단계에서는 `createSurfaceUpdate`만 전송한다.
4. 테스트 코드는 초기 렌더링 직후 동기적으로 초기 상태를 검증한 다음, `waitFor`로 비동기 상태 전환 완료를 기다리며 업데이트된 속성이 DOM에 반영되었는지 확인한다.
