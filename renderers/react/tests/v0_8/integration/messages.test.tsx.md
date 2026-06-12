# renderers/react/tests/v0_8/integration/messages.test.tsx

## 개요

A2UI React 렌더러(v0_8)의 메시지 처리 통합 테스트 파일이다. `processMessages` / `clearSurfaces` API를 통해 서버-클라이언트 메시지(`surfaceUpdate`, `beginRendering`, `deleteSurface`)가 올바르게 처리되는지를 검증한다. 단일 서피스·다중 서피스·서피스 삭제·서피스 초기화 등 네 가지 시나리오로 구성되어 있으며, React Testing Library를 사용해 실제 DOM 렌더링 결과를 단언한다.

## 의존성

### 외부 패키지
- `vitest` — `describe`, `it`, `expect` 테스트 프레임워크
- `@testing-library/react` — `render`, `screen`, `waitFor`
- `react` — `React`, `useEffect`, `useState`
- `@a2ui/web_core/types/types` — `Types.ServerToClientMessage` 타입 (타입 전용 import)

### 저장소 내부 모듈
- [`../../../src/v0_8`](../../../src/v0_8.md) — `A2UIProvider`, `A2UIRenderer`, `useA2UI` (핵심 렌더러 공개 API)
- [`../utils`](../utils/index.ts.md) — `TestWrapper`, `TestRenderer`, `createSurfaceUpdate`, `createBeginRendering`, `createDeleteSurface` (테스트 헬퍼)

## Exports

이 파일은 테스트 파일이므로 외부로 내보내는 항목이 없다.

## 테스트 케이스 상세 명세

### describe: `Message Processing`

최상위 그룹. 하위에 네 개의 `describe` 블록을 포함한다.

---

#### describe: `Basic Processing`

##### `should not render surface until beginRendering is received`

- **검증 동작**: `surfaceUpdate` 메시지만 전달했을 때 컨텐츠가 DOM에 나타나지 않아야 하며, 이후 `beginRendering`이 도착하면 비로소 컨텐츠가 렌더링되어야 한다.
- **픽스처/모킹**: `StagedRenderer` 인라인 컴포넌트를 정의한다. `useA2UI()`에서 `processMessages`를 꺼내고 `stage` 상태(`'initial' | 'updated' | 'rendering'`)로 단계를 제어한다.
  - `stage === 'initial'` 일 때: `createSurfaceUpdate`로 `text-1`(텍스트 `'Should not appear yet'`) 메시지만 전송하고 `stage`를 `'updated'`로 전환한다.
  - `stage === 'updated'` 일 때: 10 ms `setTimeout` 후 `createBeginRendering('text-1')`을 전송하고 `stage`를 `'rendering'`으로 전환한다.
- **단언**: 동기 시점에 `stage` 텍스트가 `'updated'`이고 `'Should not appear yet'` 텍스트가 DOM에 없음을 확인한다. `waitFor`로 비동기 완료 후 `stage`가 `'rendering'`이고 해당 텍스트가 DOM에 있음을 확인한다.

##### `should process surfaceUpdate and beginRendering messages`

- **검증 동작**: `surfaceUpdate` + `beginRendering`을 함께 배열로 전달하면 즉시 컨텐츠가 렌더링된다.
- **픽스처/모킹**: `TestWrapper` + `TestRenderer` 조합을 사용한다. `messages` 배열에 `createSurfaceUpdate`(id `'text-1'`, 텍스트 `'Hello World'`)와 `createBeginRendering('text-1')`을 담아 `TestRenderer`에 전달한다.
- **단언**: `'Hello World'` 텍스트가 DOM에 있음을 확인한다.

##### `should process multiple messages in sequence`

- **검증 동작**: `processMessages`를 연속으로 두 번 호출했을 때 마지막 상태만 렌더링된다(이전 상태는 사라진다).
- **픽스처/모킹**: `SequentialRenderer` 인라인 컴포넌트를 정의한다. `useEffect` 내에서 첫 번째 호출로 `'Initial'` 텍스트를 렌더링하고, 두 번째 호출로 동일 id `'text-1'`에 `'Updated'` 텍스트를 덮어쓴다. 각 호출에 `createBeginRendering`을 포함한다.
- **단언**: `'Updated'`가 DOM에 있고 `'Initial'`은 없음을 확인한다.

##### `should handle empty message arrays gracefully`

- **검증 동작**: 빈 배열 `[]`을 먼저 전달해도 오류 없이 동작하고, 이후 유효한 메시지를 처리할 수 있어야 한다.
- **픽스처/모킹**: `EmptyMessagesRenderer` 인라인 컴포넌트. `useEffect`에서 `processMessages([])`를 먼저 호출한 뒤, 바로 `createSurfaceUpdate`(id `'text-1'`, 텍스트 `'After empty'`) + `createBeginRendering('text-1')`을 전달한다.
- **단언**: `'After empty'`가 DOM에 있음을 확인한다.

---

#### describe: `Multiple Surfaces`

##### `should render different content on different surfaces`

- **검증 동작**: `surfaceId`가 다른 두 `A2UIRenderer` 인스턴스가 각자의 서피스 데이터만 표시해야 한다.
- **픽스처/모킹**: `MultiSurfaceRenderer` 인라인 컴포넌트. `useEffect`에서 단일 `processMessages` 호출로 `'surface-a'`(텍스트 `'Surface A Content'`)와 `'surface-b'`(텍스트 `'Surface B Content'`) 두 서피스를 모두 설정한다. 렌더에서 각 `A2UIRenderer`를 `data-testid`가 있는 `div`로 감싼다.
- **단언**: 두 텍스트가 모두 DOM에 있고, 각 텍스트가 해당 서피스 컨테이너 `div` 안에 위치함을 `toContainElement`로 확인한다.

##### `should update surfaces independently`

- **검증 동작**: 한 서피스를 업데이트할 때 다른 서피스는 변경되지 않아야 한다.
- **픽스처/모킹**: `IndependentSurfaceRenderer` 인라인 컴포넌트. `step` 상태로 제어한다.
  - `step === 0`: `'surface-a'`(`'A: Initial'`)와 `'surface-b'`(`'B: Initial'`)를 초기화하고 `step`을 1로 전환한다.
  - `step === 1`: `'surface-a'`만 `'A: Updated'`로 업데이트하고 `createBeginRendering`도 함께 전송한다.
- **단언**: `'A: Updated'`와 `'B: Initial'`이 모두 DOM에 있음을 확인한다.

##### `should render nothing for non-existent surface`

- **검증 동작**: 아직 데이터가 없는 `surfaceId`로 `A2UIRenderer`를 렌더링해도 오류가 발생하지 않아야 한다.
- **픽스처/모킹**: `surfaceId="does-not-exist"`인 `A2UIRenderer`를 `A2UIProvider` 안에 직접 렌더링한다. 메시지를 전혀 전달하지 않는다.
- **단언**: 암묵적으로 에러 없이 렌더링되는 것만 확인한다(명시적 단언 없음).

---

#### describe: `Delete Surface`

##### `should remove surface content when deleteSurface is received`

- **검증 동작**: `createDeleteSurface`로 특정 서피스를 삭제하면 해당 서피스 컨텐츠가 DOM에서 사라져야 한다.
- **픽스처/모킹**: `DeleteSurfaceRenderer` 인라인 컴포넌트. `deleted` boolean 상태로 삭제 완료를 추적한다.
  - `useEffect` 즉시 실행부: `'deletable-surface'`를 생성·렌더링한다.
  - 10 ms 후: `createDeleteSurface('deletable-surface')`를 전송하고 `deleted`를 `true`로 설정하며, `data-testid="deleted-marker"` span을 표시한다.
- **단언**: 동기 시점에 `'Surface content'`가 DOM에 있음을 확인한 뒤, `waitFor`로 `deleted-marker`가 나타나고 `'Surface content'`가 사라짐을 확인한다.

##### `should handle deleting a non-existent surface gracefully`

- **검증 동작**: 존재하지 않는 서피스에 `deleteSurface`를 보내도 예외가 발생하지 않아야 한다.
- **픽스처/모킹**: `DeleteNonExistentRenderer` 인라인 컴포넌트. `useEffect`에서 `createDeleteSurface('does-not-exist')`를 즉시 전송하고 `attempted`를 `true`로 설정한다. `data-testid="status"` span에 상태 텍스트를 표시한다.
- **단언**: `status` 요소의 텍스트 내용이 `'completed'`임을 확인한다.

##### `should only delete the specified surface, leaving others intact`

- **검증 동작**: 여러 서피스 중 하나만 삭제할 경우 삭제된 서피스만 사라지고 나머지는 그대로여야 한다.
- **픽스처/모킹**: `MultiSurfaceDeleteRenderer` 인라인 컴포넌트. 초기에 `'surface-a'`(`'Surface A content'`)와 `'surface-b'`(`'Surface B content'`)를 함께 생성한다. 10 ms 후 `createDeleteSurface('surface-a')`만 전송한다.
- **단언**: 동기 시점에 두 텍스트가 모두 DOM에 있음을 확인한 뒤, `waitFor`로 `deleted-marker`가 나타나고 `'Surface A content'`는 없고 `'Surface B content'`는 있음을 확인한다.

##### `should allow re-creating a surface after deletion with the same ID`

- **검증 동작**: 삭제된 서피스 ID를 재사용하여 새 컨텐츠로 다시 생성할 수 있어야 한다.
- **픽스처/모킹**: `RecreateAfterDeleteRenderer` 인라인 컴포넌트. `stage` 상태(`'initial' | 'deleted' | 'recreated'`)로 세 단계를 관리한다.
  - `'initial'`: `'recyclable-surface'`에 `'Original content'`를 생성하고 10 ms 후 `stage`를 `'deleted'`로 전환한다.
  - `'deleted'`: `createDeleteSurface('recyclable-surface')`를 전송하고 10 ms 후 `'recreated'`로 전환한다.
  - `'recreated'`: 동일 서피스 ID에 id `'text-2'`, 텍스트 `'New content after recreation'`으로 재생성한다.
- **단언**: 초기에 `'Original content'`가 DOM에 있음을 확인한 뒤, `waitFor`로 `stage`가 `'recreated'`이고 `'Original content'`는 없으며 `'New content after recreation'`이 있음을 확인한다.

---

#### describe: `Clear Surfaces`

##### `should clear all surfaces when clearSurfaces is called`

- **검증 동작**: `clearSurfaces()`를 호출하면 모든 서피스 컨텐츠가 DOM에서 제거되어야 한다.
- **픽스처/모킹**: `ClearRenderer` 인라인 컴포넌트. `useA2UI()`에서 `processMessages`와 `clearSurfaces`를 함께 꺼낸다. `useEffect`에서 먼저 `'Will be cleared'` 텍스트를 렌더링하고, 10 ms 후 `clearSurfaces()`를 호출하며 `cleared`를 `true`로 설정한다.
- **단언**: 동기 시점에 `'Will be cleared'`가 DOM에 있음을 확인한 뒤, `waitFor`로 `cleared-marker`가 나타나고 텍스트가 사라짐을 확인한다.

##### `should allow new content after clearing`

- **검증 동작**: `clearSurfaces()` 후에도 새 `processMessages` 호출로 컨텐츠를 다시 렌더링할 수 있어야 한다.
- **픽스처/모킹**: `ClearAndRefillRenderer` 인라인 컴포넌트. `step` 상태(0 → 1 → 2)로 흐름을 제어한다.
  - `step === 0`: `'Original'` 텍스트를 렌더링하고 `step`을 1로 전환한다.
  - `step === 1`: `clearSurfaces()`를 호출하고 `step`을 2로 전환한다.
  - `step === 2`: id `'text-2'`, 텍스트 `'New Content'`로 새 컨텐츠를 렌더링한다.
- **단언**: `waitFor`로 `'New Content'`가 DOM에 있고 `'Original'`이 없음을 확인한다.

## 동작 흐름

이 파일은 실행 로직을 직접 갖지 않으며, 각 테스트가 독립적으로 `A2UIProvider`를 마운트하고 인라인 컴포넌트 또는 `TestWrapper` + `TestRenderer` 조합을 통해 메시지를 주입한 뒤 DOM 상태를 단언하는 패턴을 따른다. 비동기 타이밍이 필요한 케이스는 `setTimeout` + `waitFor`를 조합하여 단계를 분리한다. 모든 테스트는 동일한 `A2UIProvider` 컨텍스트 경계 안에서 동작하므로 각 테스트가 새로운 Provider 인스턴스를 사용하여 상태 격리가 보장된다.
