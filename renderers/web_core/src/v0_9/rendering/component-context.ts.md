# renderers/web_core/src/v0_9/rendering/component-context.ts

## 개요

렌더링 시점에 개별 컴포넌트에 제공되는 컨텍스트 객체를 정의한다. `ComponentContext`는 컴포넌트 모델, 데이터 컨텍스트, 서피스 컴포넌트 목록, 테마에 대한 접근을 단일 인터페이스로 통합하며, 액션 디스패치 메커니즘을 제공한다.

## 의존성

### 저장소 내부 모듈
- [`./data-context.js`](./data-context.ts.md) — `DataContext`
- [`../state/component-model.js`](../state/component-model.ts.md) — `ComponentModel`
- [`../state/surface-model.js`](../state/surface-model.ts.md) — `SurfaceModel` (타입 전용)
- [`../state/surface-components-model.js`](../state/surface-components-model.ts.md) — `SurfaceComponentsModel` (타입 전용)
- [`../errors.js`](../errors.ts.md) — `A2uiStateError`

## Exports

| 이름 | 종류 |
|------|------|
| `ComponentContext` | 클래스 |

## 상세 명세

### 클래스 `ComponentContext`

렌더러가 컴포넌트를 렌더링할 때 주입받는 불변(readonly) 컨텍스트 객체이다.

#### 필드

| 필드 | 타입 | 설명 |
|------|------|------|
| `componentModel` | `ComponentModel` (readonly) | 이 컴포넌트의 id, 타입, 속성을 담은 상태 모델 |
| `dataContext` | `DataContext` (readonly) | `dataModelBasePath`를 기준으로 스코프된 데이터 컨텍스트 |
| `surfaceComponents` | `SurfaceComponentsModel` (readonly) | 현재 서피스의 전체 컴포넌트 맵 (id로 조회 가능) |
| `theme` | `any` (readonly) | 서피스에 설정된 테마 객체; 지정되지 않으면 빈 객체(`{}`) |
| `_actionDispatcher` | `(action: any) => Promise<void>` (private) | 클로저로 캡처된 서피스-바운드 액션 디스패처 |

#### 생성자 `constructor(surface: SurfaceModel<any>, componentId: string, dataModelBasePath: string = '/')`

1. `surface.componentsModel.get(componentId)`로 컴포넌트 모델을 조회한다.
2. 컴포넌트가 없으면 `A2uiStateError` `"Component not found: {componentId}"`를 던진다.
3. `this.componentModel`, `this.surfaceComponents`, `this.theme`에 각각 조회된 모델, `surface.componentsModel`, `surface.theme`을 할당한다.
4. `new DataContext(surface, dataModelBasePath)`로 데이터 컨텍스트를 생성한다.
5. `this._actionDispatcher`를 `action => surface.dispatchAction(action, this.componentModel.id)` 클로저로 설정한다.

#### `dispatchAction(action: any): Promise<void>`

`this._actionDispatcher(action)`을 호출하고 결과 Promise를 반환한다. 내부적으로 `SurfaceModel.dispatchAction()`에 현재 컴포넌트의 `id`를 `sourceComponentId`로 전달하여 액션을 서피스 수준에서 발행한다.

## 동작 흐름

렌더러는 각 컴포넌트를 렌더링하기 직전에 `new ComponentContext(surface, componentId, basePath)`를 생성한다. 생성 실패(존재하지 않는 컴포넌트)는 즉시 오류로 처리된다. 컴포넌트는 `dataContext`를 통해 데이터를 읽거나 구독하고, `dispatchAction()`으로 사용자 이벤트를 서피스에 전달한다. 데이터 경로는 `dataModelBasePath`를 기준으로 상대 경로 해석이 이루어진다.
