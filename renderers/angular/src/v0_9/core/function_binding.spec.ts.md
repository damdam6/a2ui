# renderers/angular/src/v0_9/core/function_binding.spec.ts

## 개요

`@a2ui/web_core/v0_9`의 함수 바인딩 기능(`add`, `formatString`)이 Angular Signal 변환 레이어를 통해 올바르게 동작하는지 검증하는 테스트 파일이다. `DataContext.resolveSignal`로 반환된 Preact Signal을 `toAngularSignal`로 Angular Signal로 변환한 뒤, 데이터 모델 업데이트에 따라 리액티브하게 값이 갱신되는 통합 동작을 확인한다. 총 3개의 테스트 케이스가 2개의 `describe` 블록(`add`, `formatString`)으로 구성된다.

## 의존성

### 외부 패키지
- `@a2ui/web_core/v0_9` — `DataContext`, `SurfaceModel`
- `@angular/core` — `DestroyRef`

### 저장소 내부 모듈
- [`../catalog/basic/basic-catalog`](../catalog/basic/basic-catalog.ts.md) — `BasicCatalogBase`
- [`./utils`](./utils.ts.md) — `toAngularSignal`

## Exports

없음 (테스트 전용 파일).

## 테스트 케이스 명세

### 공통 픽스처 (`beforeEach`)

`jasmine.createSpyObj`로 `DestroyRef` 목 객체(`mockDestroyRef`)를 생성한다. `onDestroy` 스파이는 호출 시 빈 해제 함수 `() => {}`를 반환하도록 설정된다.

---

### `describe('add')`

#### `it('should update output correctly when bound input updates using function call binding')`

**검증 동작**: `add` 함수 바인딩을 통해 데이터 모델의 `/inputValue` 경로가 변경될 때 Angular Signal이 `inputValue + 1` 값으로 리액티브하게 갱신되는지 확인한다.

**픽스처/모킹**:
- `BasicCatalogBase` 인스턴스를 카탈로그로 사용.
- `SurfaceModel('surface_1', catalog)`로 서피스를 생성하고, `DataContext(surface, '/')` 로 컨텍스트를 생성.
- `callValue` 객체: `call: 'add'`, `args: { a: { path: '/inputValue' }, b: 1 }`, `returnType: 'number'`

**검증 단계**:
1. `context.resolveSignal<number>(callValue as any)` → Preact Signal 반환.
2. `toAngularSignal(resSig, mockDestroyRef)` → Angular Signal 생성.
3. 초기 상태: `angSig()`는 `NaN`이어야 한다 (`/inputValue`가 미정의이므로).
4. `dataModel.set('/inputValue', 5)` 후 `angSig()`는 `6`이어야 한다.
5. `dataModel.set('/inputValue', 10)` 후 `angSig()`는 `11`이어야 한다.

---

### `describe('formatString')`

#### `it('should correctly format string with dynamic path and dollar sign')`

**검증 동작**: `formatString` 바인딩에서 `'$${/price}'` 패턴이 달러 기호를 유지하면서 경로 값을 올바르게 보간하는지 확인한다. 이전 회귀 버그(Signal 객체가 `[object Object]`로 직렬화되던 문제)의 재발도 방지한다.

**픽스처/모킹**:
- `callValue`: `call: 'formatString'`, `args: { value: '$${/price}' }`, `returnType: 'string'`

**검증 단계**:
1. 초기 상태: `angSig()`는 `'$'`이어야 한다 (`/price`가 미정의 → 빈 문자열로 치환).
2. `dataModel.set('/price', 42)` 후 `angSig()`는 `'$42'`이어야 한다.
3. `typeof angSig()`는 `'string'`이어야 한다.

#### `it('should handle multiple path interpolations correctly')`

**검증 동작**: `formatString` 바인딩에서 여러 경로 보간(`${/firstName} ${/lastName}`)이 올바르게 합쳐지는지 확인한다.

**검증 단계**:
1. `dataModel.set('/firstName', 'A2UI')`, `dataModel.set('/lastName', 'Renderer')` 설정.
2. `angSig()`는 `'A2UI Renderer'`이어야 한다.

## 동작 흐름

각 테스트는 독립적인 `SurfaceModel` + `DataContext`를 생성하고, `resolveSignal`을 통해 함수 바인딩이 담긴 Preact Signal을 얻은 후 `toAngularSignal`로 Angular Signal로 변환한다. 이후 `dataModel.set()`을 호출하여 데이터 모델을 업데이트하고, Angular Signal의 값이 기대값과 일치하는지 즉시 동기적으로 검증한다.
