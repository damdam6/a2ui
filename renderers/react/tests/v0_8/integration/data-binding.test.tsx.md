# renderers/react/tests/v0_8/integration/data-binding.test.tsx

## 개요

`dataModelUpdate` 메시지 처리와 경로(path) 바인딩 동작을 검증하는 통합 테스트 파일이다. 서버에서 전송하는 데이터 모델 업데이트가 여러 컴포넌트에 올바르게 반영되는지, 그리고 같은 경로를 공유하는 컴포넌트들이 동기적으로 갱신되는지를 확인한다. 총 3개의 `describe` 그룹, 16개 테스트 케이스로 구성된다.

## 의존성

### 외부 패키지
- `vitest` — `describe`, `it`, `expect`
- `@testing-library/react` — `render`, `screen`, `fireEvent`, `waitFor`
- `react` — `React`, `useEffect`

### 저장소 내부 모듈
- [`../../../src/v0_8`](../../../src/v0_8.md) — `A2UIProvider`, `A2UIRenderer`, `useA2UI`
- `@a2ui/web_core/types/types` — `Types` (타입 전용 import, 외부 패키지이지만 모노레포 내부 패키지)
- [`../utils`](../utils/index.ts.md) — `TestWrapper`, `TestRenderer`, `createSurfaceUpdate`, `createBeginRendering`, `createDataModelUpdate`

## Exports

이 파일은 아무것도 export하지 않는다. `describe` 블록이 직접 vitest 런타임에 등록된다.

## 테스트 케이스 상세 명세

### describe: `Data Binding`

#### describe: `dataModelUpdate Messages`

**`should initialize data model via dataModelUpdate before rendering`**
- 검증 동작: `createDataModelUpdate`로 `greeting` 키에 `'Hello from data model'` 값을 초기화한 뒤, 해당 path를 바인딩한 `Text` 컴포넌트를 렌더링했을 때 그 문자열이 DOM에 나타나는지 확인한다.
- 픽스처/모킹: 인라인 컴포넌트 `DataModelRenderer`가 `useEffect` 안에서 `processMessages`를 호출해 `dataModelUpdate → surfaceUpdate → beginRendering` 순으로 메시지를 처리한다. `A2UIProvider`로 래핑.
- 검증 방식: `screen.getByText('Hello from data model')` 동기 단언.

**`should update existing data model values`**
- 검증 동작: 초기에 `'Count: 0'`으로 설정된 `counter` path가 `setTimeout(10ms)` 후 `'Count: 1'`로 업데이트될 때 DOM이 갱신되는지 확인한다.
- 픽스처/모킹: `DataUpdateRenderer` — `React.useState(0)`으로 단계(0→1)를 관리. step 0에서 초기 메시지 배치를 처리하고 `setStep(1)` 호출, step 1에서 10ms 지연 후 업데이트 메시지를 전송.
- 검증 방식: `waitFor`로 비동기 단언.

---

#### describe: `Path Bindings`

**`should propagate data changes across components sharing the same path`**
- 검증 동작: `shared.name` path를 공유하는 `TextField`와 `Text` 컴포넌트에서, `input` 요소에 `fireEvent.change`로 `'Alice'` 입력 시 `Text` 컴포넌트에도 즉시 반영되는지 확인한다.
- 픽스처/모킹: `SharedDataRenderer` — `Column` > `[TextField, Text]` 구조. 초기 데이터 모델 없이 path 바인딩만 설정.
- 검증 방식: `container.querySelector('input')` 후 `fireEvent.change`, `waitFor`로 비동기 단언.

**`should handle TextField value binding`**
- 검증 동작: `form.username` path를 바인딩한 `TextField`에서 `fireEvent.change`로 `'testuser'` 입력 후 `input.value`가 그 값을 유지하는지 확인한다.
- 픽스처/모킹: `TestWrapper` + `TestRenderer` 사용. `messages` 배열을 직접 선언(`Types.ServerToClientMessage[]`).
- 검증 방식: `expect(input).toHaveValue('testuser')` 동기 단언.

**`should handle CheckBox checked binding via path`**
- 검증 동작: `form.agree` path를 바인딩한 `CheckBox`가 초기에 `false`(unchecked)이고, `fireEvent.click` 후 `true`(checked)로 전환되는지 확인한다.
- 픽스처/모킹: `TestWrapper` + `TestRenderer`. `container.querySelector('input[type="checkbox"]')`로 체크박스 요소 획득.
- 검증 방식: `checkbox.checked` 동기 단언.

---

#### describe: `Server-Driven Data Model Updates`

이 그룹의 테스트들은 모두 동일한 패턴을 따른다: `stage` state (`'initial' | 'updated'`)로 렌더링 단계를 관리하고, `setTimeout(10ms)` 후 두 번째 `processMessages` 호출로 데이터 모델을 업데이트한 뒤 `waitFor`로 DOM 갱신을 검증한다. 각 테스트는 `data-testid="stage"` 요소로 단계 진행 여부를 추가 확인한다.

**`should update TextField when dataModelUpdate changes bound path value`**
- path: `user.name`, 초기값: `'Alice'`, 업데이트값: `'Bob'`
- `TextField` (text 프로퍼티 바인딩), `input` 요소의 value 확인.

**`should update CheckBox when dataModelUpdate changes bound path value`**
- path: `settings.enabled`, 초기값: `valueBoolean: false`, 업데이트값: `valueBoolean: true`
- `CheckBox` (value 프로퍼티 바인딩), `getByRole('checkbox')` 사용, `.not.toBeChecked()` → `.toBeChecked()` 전환 검증.

**`should update Slider when dataModelUpdate changes bound path value`**
- path: `volume`, 초기값: `valueNumber: 30`, 업데이트값: `valueNumber: 80`
- `Slider` (value 프로퍼티, minValue: 0, maxValue: 100), `getByRole('slider').toHaveValue('30')` → `'80'` 검증.

**`should update multiple components bound to the same path simultaneously`**
- path: `shared.value`, 초기값: `'Initial'`, 업데이트값: `'Updated'`
- `Text` + `TextField`(Column 자식)가 동시에 갱신되는지 확인. `getByText('Updated')` + `input.toHaveValue('Updated')` 모두 검증.

**`should handle dataModelUpdate with multiple key/value pairs at once`**
- path: `form.firstName` / `form.lastName`, 초기값: `'John'` / `'Doe'`, 업데이트값: `'Jane'` / `'Smith'`
- 단일 `createDataModelUpdate` 호출에 복수 항목 전달. 이전 값 소멸(`queryByText('John')` not in document)도 함께 검증.
- Row 레이아웃: `[text-first, text-last]`.

**`should handle deeply nested path values in dataModelUpdate`**
- path: `user.profile.address.city`, 초기값: `'New York'`, 업데이트값: `'Los Angeles'`
- 4단계 중첩 경로 처리 검증. 이전 텍스트 소멸도 확인.

**`should update DateTimeInput when dataModelUpdate changes bound path value`**
- path: `appointment.date`, 초기값: `'2024-01-15'`, 업데이트값: `'2024-06-20'`
- `DateTimeInput` (value 바인딩, enableDate: true, enableTime: false). `container.querySelector('input[type="date"]')`로 요소 획득, value 문자열 직접 검증.

**`should update MultipleChoice when dataModelUpdate changes bound path value`**
- path: `survey.answers` (selections 바인딩)
- options 3개: `{label: 'Option 1', value: 'option1'}` 등. `<select>` 요소와 3개 `<option>`의 존재 및 value 검증. 업데이트 단계 없음 — 렌더링 후 즉시 select 구조만 확인.

**`should update Image when dataModelUpdate changes bound path value`**
- path: `product.imageUrl`, 초기값: `'https://example.com/old-product.jpg'`, 업데이트값: `'https://example.com/new-product.jpg'`
- `Image` (url 바인딩, usageHint: `'mediumFeature'`). `<img>` 요소의 `src` 어트리뷰트 검증.

**`should update Icon when dataModelUpdate changes bound path value`**
- path: `ui.statusIcon`, 초기값: `'check'`, 업데이트값: `'error'`
- `Icon` (name 바인딩). `container.querySelector('.g-icon')` 요소의 텍스트 콘텐츠로 아이콘명 검증.

**`should update Video when dataModelUpdate changes bound path value`**
- path: `media.videoUrl`, 초기값: `'https://example.com/old-video.mp4'`, 업데이트값: `'https://example.com/new-video.mp4'`
- `Video` (url 바인딩). `<video>` 요소의 `src` 어트리뷰트 검증.

**`should update AudioPlayer when dataModelUpdate changes bound path value`**
- path: `media.audioUrl`, 초기값: `'https://example.com/old-audio.mp3'`, 업데이트값: `'https://example.com/new-audio.mp3'`
- `AudioPlayer` (url 바인딩, description: `'Audio track'`). `<audio>` 요소의 `src` 어트리뷰트 검증.

## 동작 흐름

모든 테스트는 다음 흐름 중 하나를 따른다.

**단순 동기 흐름**: `render(A2UIProvider > SomeRenderer)` → `useEffect`에서 `processMessages([dataModelUpdate, surfaceUpdate, beginRendering])` 일괄 처리 → 동기 DOM 단언.

**2단계 비동기 흐름**: `stage = 'initial'` 상태에서 초기 메시지 배치 처리 + `setTimeout(10ms)` 후 `stage = 'updated'`로 전환 → `stage = 'updated'`에서 `processMessages([dataModelUpdate])` 호출 → `waitFor`로 DOM 갱신 비동기 단언. `data-testid="stage"` 요소를 추가 신호로 사용해 단계 전환 완료 여부를 확인한다.

**사용자 인터랙션 흐름**: `render` → `fireEvent.change` 또는 `fireEvent.click` → 동기 또는 `waitFor` 비동기 단언으로 상태 반영 확인.
