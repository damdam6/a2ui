# renderers/web_core/src/v0_9/rendering/component-context.test.ts

## 개요

`ComponentContext` 클래스의 초기화, 액션 디스패치, 오류 처리, 데이터 컨텍스트 경로, 테마 노출 동작을 검증하는 단위 테스트 파일이다. Node.js 내장 테스트 러너(`node:test`)를 사용한다.

## 의존성

### 외부 패키지
- `node:test` — `describe`, `it`
- `node:assert` — `assert`

### 저장소 내부 모듈
- [`./component-context.js`](./component-context.ts.md) — 테스트 대상 `ComponentContext`
- [`../state/surface-model.js`](../state/surface-model.ts.md) — `SurfaceModel`
- [`../state/component-model.js`](../state/component-model.ts.md) — `ComponentModel`

## 픽스처

`describe('ComponentContext')` 블록 상단에 공유 픽스처를 선언한다:
- `mockSurface`: `new SurfaceModel('surface1', {} as any)` — 카탈로그 없이 생성한 서피스 목(mock)
- `componentId`: `'comp1'`
- `componentModel`: `new ComponentModel('comp1', 'TestComponent', {})` — 서피스에 `addComponent()`로 등록됨

## 테스트 케이스

### `initializes correctly`
- 동작: `new ComponentContext(mockSurface, componentId)`를 생성한다.
- 검증:
  - `context.componentModel === componentModel` (동일 참조)
  - `context.dataContext`가 truthy
  - `context.surfaceComponents === mockSurface.componentsModel` (동일 참조)

### `dispatches actions`
- 동작: `new ComponentContext(mockSurface, componentId)` 생성 후, `mockSurface.onAction`을 구독하여 수신된 액션을 `actionDispatched`에 저장하고, `context.dispatchAction({ event: { name: 'test', context: { a: 1 } } })`를 호출한다.
- 검증:
  - `actionDispatched.name === 'test'`
  - `actionDispatched.sourceComponentId === componentId`
  - `actionDispatched.context` deepEqual `{ a: 1 }`
- 정리: 테스트 종료 후 `subscription.unsubscribe()` 호출

### `throws error if component not found`
- 동작: `new ComponentContext(mockSurface, 'nonExistentId')` 생성을 시도한다.
- 검증: `/Component not found: nonExistentId/` 오류가 발생함을 확인한다.

### `creates data context with correct base path`
- 동작: `new ComponentContext(mockSurface, componentId, '/foo/bar')`를 생성한다.
- 검증: `context.dataContext.path === '/foo/bar'`임을 확인한다.

### `exposes theme from surface`
- 픽스처: `theme = { primaryColor: '#FF5733' }`를 가진 `new SurfaceModel('themed', {} as any, theme)`와 `new ComponentModel('c1', 'Text', {})`를 해당 서피스에 등록
- 동작: `new ComponentContext(themedSurface, 'c1')` 생성
- 검증:
  - `context.theme` deepEqual `{ primaryColor: '#FF5733' }`
  - `context.theme.primaryColor === '#FF5733'`

### `exposes empty theme when none provided`
- 동작: 테마를 지정하지 않은 `mockSurface`로 컨텍스트 생성
- 검증: `context.theme` deepEqual `{}`

## 동작 흐름

테스트는 공유 `mockSurface`와 `componentModel`을 모든 케이스에서 재사용하며, 각 테스트는 독립적인 `ComponentContext` 인스턴스를 생성한다. 액션 디스패치 테스트는 `onAction` 이벤트 소스를 직접 구독하여 서피스 수준에서 검증하고, 테스트 완료 후 구독을 해제한다.
