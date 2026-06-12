# renderers/react/tests/v0_8/integration/hooks.test.tsx

## 개요

`useA2UI` React 컨텍스트 훅의 동작과 안정성을 검증하는 통합 테스트 파일이다. Provider 외부에서 훅을 사용할 때 에러가 발생하는지, 그리고 여러 번 렌더링 시 반환되는 API 참조(특히 `processMessages`)가 안정적으로 동일 참조를 유지하는지를 확인한다. 총 1개의 `describe` 그룹, 2개 테스트 케이스로 구성된다.

## 의존성

### 외부 패키지
- `vitest` — `describe`, `it`, `expect`, `vi`
- `@testing-library/react` — `render`
- `react` — `React`

### 저장소 내부 모듈
- [`../../../src/v0_8`](../../../src/v0_8.md) — `A2UIProvider`, `useA2UI`
- [`../utils`](../utils/index.ts.md) — `getElement`

## Exports

이 파일은 아무것도 export하지 않는다. `describe` 블록이 직접 vitest 런타임에 등록된다.

## 테스트 케이스 상세 명세

### describe: `Context Hooks`

**`should throw error when useA2UI is used outside provider`**
- 검증 동작: `A2UIProvider`로 감싸지 않은 컴포넌트 안에서 `useA2UI()`를 호출하면 렌더링 시 에러가 throw되는지 확인한다.
- 픽스처/모킹: `vi.spyOn(console, 'error').mockImplementation(() => {})` — React가 에러 발생 시 `console.error`를 호출하는데, 이를 억제해 테스트 출력이 오염되지 않게 한다. 테스트 완료 후 `consoleSpy.mockRestore()`로 복원. 인라인 컴포넌트 `BadComponent`는 훅만 호출하고 `null`을 반환.
- 검증 방식: `expect(() => render(<BadComponent />)).toThrow()` — 에러 종류는 명시하지 않고 throw 여부만 단언.

**`should provide stable action references across renders`**
- 검증 동작: `useA2UI()`가 렌더 사이클마다 동일한 `processMessages` 함수 참조를 반환하는지(참조 동일성 유지) 확인한다.
- 픽스처/모킹: 모듈 레벨 배열 `actionRefs: Array<ReturnType<typeof useA2UI>>` 에 각 렌더 시 `useA2UI()`의 반환값을 누적 저장. `ActionTracker` 컴포넌트는 `api.processMessages`를 `onClick`에 연결한 버튼을 렌더링. `render` 후 `rerender`로 동일 컴포넌트를 한 번 더 렌더링해 두 개의 참조를 수집.
- 검증 방식: `getElement(actionRefs, 0)`과 `getElement(actionRefs, 1)`로 첫 번째·두 번째 참조를 꺼낸 뒤 `expect(ref0.processMessages).toBe(ref1.processMessages)`로 참조 동일성(`===`) 단언.

## 동작 흐름

두 테스트는 독립적이다.

첫 번째 테스트는 에러 경계 동작을 확인한다. `console.error`를 모킹 후 Provider 없이 렌더링을 시도하고, 에러가 발생하는지 단언한 뒤 모킹을 해제한다.

두 번째 테스트는 참조 안정성을 확인한다. 초기 렌더 → `rerender`로 재렌더 → 수집된 두 API 객체의 `processMessages` 프로퍼티가 `===`인지 단언한다. 이는 `useA2UI`가 내부적으로 `useCallback` 또는 이와 동등한 메모이제이션을 적용해 함수 참조를 안정적으로 유지해야 함을 요구 조건으로 명시한다.
