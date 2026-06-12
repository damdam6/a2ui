# renderers/web_core/src/v0_9/test/test-utils.ts

## 개요

컴포넌트 렌더링 계층의 단위/통합 테스트에서 재사용되는 공용 픽스처 유틸리티 파일이다. 최소한의 설정으로 `ComponentContext`를 생성할 수 있는 헬퍼 함수와, 테스트 전용 `SurfaceModel` 서브클래스를 제공한다. 소스 파일을 수정하지 않고 독립적인 테스트 환경을 구성하기 위해 존재한다.

## 의존성

### 외부 패키지
없음.

### 저장소 내부 모듈
- [`../rendering/component-context.ts`](../rendering/component-context.ts.md) — `ComponentContext`
- [`../state/surface-model.ts`](../state/surface-model.ts.md) — `SurfaceModel`
- [`../catalog/types.ts`](../catalog/types.ts.md) — `Catalog`, `ComponentApi`
- [`../state/component-model.ts`](../state/component-model.ts.md) — `ComponentModel`

## Exports

| 이름 | 종류 |
|---|---|
| `TestSurfaceModel` | 클래스 (exported) |
| `createTestContext` | 함수 (exported) |

## 상세 명세

### 클래스: `TestSurfaceModel`

**선언:**
```
export class TestSurfaceModel extends SurfaceModel<ComponentApi>
```

**생성자 시그니처:**
```
constructor(actionHandler: any = async () => {})
```

- 부모 생성자 `super('test', new Catalog('test-catalog', []), {})`를 호출한다. 첫 번째 인수는 surface 식별자 문자열 `'test'`, 두 번째 인수는 id `'test-catalog'`와 빈 배열 `[]`로 생성된 `Catalog` 인스턴스, 세 번째 인수는 빈 객체 `{}`이다.
- 부모 초기화 후 `this.onAction.subscribe(actionHandler)`를 호출하여 전달받은 `actionHandler`를 액션 이벤트에 구독시킨다.
- `actionHandler`의 기본값은 `async () => {}`(아무 것도 하지 않는 비동기 함수)이다.

**목적:** 실제 Surface 인프라 없이 테스트용 Surface를 빠르게 생성하기 위한 서브클래스. 카탈로그와 컴포넌트 목록이 비어 있으며 액션 핸들러를 외부에서 주입할 수 있다.

---

### 함수: `createTestContext`

**시그니처:**
```
export function createTestContext(properties: any, actionHandler: any = async () => {}): ComponentContext
```

**매개변수:**
- `properties: any` — 컴포넌트에 전달할 프로퍼티 객체. `ComponentModel` 생성에 사용된다.
- `actionHandler: any` — 액션 발생 시 호출될 핸들러. 기본값은 `async () => {}`.

**반환 타입:** `ComponentContext`

**동작 로직:**
1. `new TestSurfaceModel(actionHandler)`로 테스트용 surface를 생성한다.
2. `new ComponentModel('test-id', 'TestComponent', properties)`로 컴포넌트 모델을 생성한다. 고정 id `'test-id'`, 고정 타입명 `'TestComponent'`를 사용한다.
3. `surface.componentsModel.addComponent(component)`를 호출해 컴포넌트를 surface에 등록한다.
4. `new ComponentContext(surface, 'test-id', '/')`를 생성하고 반환한다. 경로는 루트 `'/'`로 고정된다.

**목적:** 테스트 코드에서 한 줄로 완전히 구성된 `ComponentContext`를 얻기 위한 팩토리 함수. 프로퍼티와 액션 핸들러만 주입하면 된다.

## 동작 흐름

파일은 두 개의 export로만 구성된다. `TestSurfaceModel`이 `SurfaceModel<ComponentApi>`를 상속해 테스트 전용 표면 모델을 제공하고, `createTestContext`는 `TestSurfaceModel` → `ComponentModel` → `surface.componentsModel` 등록 → `ComponentContext` 생성의 순서로 완전한 컴포넌트 컨텍스트를 조립해 반환한다. 두 export 모두 상태를 공유하지 않으며 각 호출마다 독립적인 인스턴스를 생성한다.
