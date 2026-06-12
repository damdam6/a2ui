# renderers/react/tests/utils.tsx

## 개요

v0_9 기반 A2UI React 컴포넌트를 격리된 환경에서 테스트하기 위한 범용 헬퍼 유틸리티 파일이다. 실제 A2UI 상태 생명주기(`SurfaceModel`, `ComponentModel`, `Catalog`)를 유지한 채 단일 컴포넌트를 렌더링할 수 있는 `renderA2uiComponent` 함수를 제공한다. 자식 컴포넌트가 카탈로그에 존재하면 실제 렌더링하고, 존재하지 않으면 플레이스홀더 `<div>`를 반환하는 스마트 `buildChild` 모킹을 포함한다.

## 의존성

### 외부 패키지
- `react` — JSX 렌더링
- `@testing-library/react` — `render` 함수
- `vitest` — `vi.fn()` 모킹
- `@a2ui/web_core/v0_9` — `SurfaceModel`, `ComponentModel`, `Catalog`, `ComponentContext`
- `@a2ui/web_core/v0_9/basic_catalog` — `BASIC_FUNCTIONS` 기본 함수 목록

### 저장소 내부 모듈
- [`../src/v0_9/adapter`](../src/v0_9/adapter.tsx.md) — `ReactComponentImplementation` 타입 (타입 전용 import)

## Exports

| 이름 | 종류 |
|---|---|
| `RenderA2uiOptions` | 인터페이스 (타입) |
| `renderA2uiComponent` | 함수 |

## 상세 명세

### `RenderA2uiOptions` 인터페이스

테스트 렌더링 시 선택적으로 전달할 수 있는 옵션 객체 타입.

| 필드 | 타입 | 기본값 | 설명 |
|---|---|---|---|
| `initialData` | `Record<string, any>` | `{}` | 데이터 모델 루트(`/`)에 설정할 초기 데이터 |
| `additionalImpls` | `ReactComponentImplementation[]` | `[]` | 자식 컴포넌트에 필요한 추가 구현체 목록 |
| `additionalComponents` | `ComponentModel[]` | `[]` | 사전에 인스턴스화된 자식 `ComponentModel` 목록 |
| `functions` | `any[]` | `BASIC_FUNCTIONS` | 카탈로그에 포함할 함수 목록 |

### `renderA2uiComponent` 함수

- **시그니처**: `renderA2uiComponent(impl: ReactComponentImplementation, componentId: string, initialProperties: Record<string, any>, options: RenderA2uiOptions = {}) → { view, surface, buildChild, mainModel, context, updateData }`
- **매개변수**:
  - `impl` — 테스트 대상 컴포넌트의 React 구현체
  - `componentId` — 컴포넌트에 부여할 고유 ID 문자열
  - `initialProperties` — 컴포넌트의 초기 프로퍼티 맵
  - `options` — 선택적 설정 (`RenderA2uiOptions` 참고)
- **동작 로직**:
  1. `options`를 구조 분해하여 각 필드에 기본값 적용(`initialData={}`, `additionalImpls=[]`, `additionalComponents=[]`, `functions=BASIC_FUNCTIONS`).
  2. `impl`과 `additionalImpls`를 합쳐 `allImpls` 배열 구성.
  3. `new Catalog('test-catalog', allImpls, functions)`로 카탈로그 생성.
  4. `new SurfaceModel<ReactComponentImplementation>('test-surface', catalog)`로 서피스 모델 생성.
  5. `surface.dataModel.set('/', initialData)`로 루트 경로에 초기 데이터 주입.
  6. `new ComponentModel(componentId, impl.name, initialProperties)`로 주 컴포넌트 모델 생성 후 `surface.componentsModel.addComponent(mainModel)`로 등록.
  7. `additionalComponents` 배열을 순회하며 각 모델을 `surface.componentsModel.addComponent`로 등록.
  8. `new ComponentContext(surface, componentId, '/')`로 루트 경로 기준 컨텍스트 생성.
  9. `buildChild`를 `vi.fn`으로 생성. `(id: string, basePath?: string)` 시그니처로 호출 시 세 가지 경로 중 하나를 택함:
     - `surface.componentsModel.get(id)`가 `null`이면 `data-testid="child-{id}"`, `data-basepath={basePath}` 속성을 가진 플레이스홀더 `<div>` 반환.
     - 모델이 존재하지만 카탈로그에서 해당 `compModel.type`의 구현체를 찾지 못하면 `data-testid="error-unknown-type-{compModel.type}"` `<div>` 반환.
     - 모델과 구현체 모두 존재하면 새 `ComponentContext(surface, id, basePath || '/')`를 생성하고 구현체의 `render` 속성을 `ChildComponent`로 호출하여 실제 렌더링.
  10. `@testing-library/react`의 `render`를 통해 `<ComponentToRender context={mainContext} buildChild={buildChild} />`를 렌더링하고 결과를 `view`에 저장.
  11. 다음 객체를 반환:
      - `view` — `@testing-library/react` render 결과
      - `surface` — `SurfaceModel` 인스턴스 (상태 직접 조작 가능)
      - `buildChild` — `vi.fn` 모킹된 buildChild 함수 (호출 여부 검증 가능)
      - `mainModel` — 주 컴포넌트 `ComponentModel` 인스턴스
      - `context` — 주 컴포넌트 `ComponentContext` 인스턴스
      - `updateData(path: string, value: any): Promise<void>` — 비동기 헬퍼. `surface.dataModel.set(path, value)` 호출 후 `new Promise(resolve => setTimeout(resolve, 0))`으로 React의 `useSyncExternalStore` 업데이트가 처리될 때까지 한 틱 대기.

## 동작 흐름

`renderA2uiComponent` 호출 → 카탈로그 및 서피스 모델 구성 → 데이터 모델 초기화 → 컴포넌트 모델 등록 → 스마트 buildChild 모킹 생성 → 실제 React 렌더링 → 테스트에서 사용할 수 있는 핸들 반환. 이후 테스트에서 `updateData`를 통해 데이터 모델을 변경하면 React가 한 틱 후 리렌더링된다.
