# renderers/angular/src/v0_9/core/component-binder.service.spec.ts

## 개요

`ComponentBinder` 서비스의 단위 테스트 파일이다. 리터럴 프로퍼티 바인딩, path 기반 양방향 바인딩, 단일 자식 참조(`child`/`trigger`/`content`), 자식 목록(`children`) 템플릿 확장, 그리고 `checks` 기반 유효성 검사 reactive 동작을 포괄적으로 검증한다.

## 의존성

### 외부 패키지
- `@angular/core/testing`: `TestBed`
- `@a2ui/web_core/v0_9`: `Catalog`, `ComponentContext`, `ComponentModel`, `SurfaceModel`

### 저장소 내부 모듈
- [`./component-binder.service`](./component-binder.service.ts.md)

## Exports

없음 (테스트 파일).

## 테스트 케이스 명세

### 헬퍼 함수: `createComponentContext`

**시그니처**: `createComponentContext({ properties, componentId?, componentType?, data? }): { context: ComponentContext; surface: SurfaceModel }`

테스트용 `SurfaceModel`과 `ComponentContext`를 구성하는 로컬 헬퍼다.
- `new Catalog('test-catalog', [])` → `new SurfaceModel('test-surface', catalog)` 순서로 생성.
- `data` 항목을 `surface.dataModel.set(path, val)`로 주입.
- `new ComponentModel(componentId, componentType, properties)`를 `surface.componentsModel.addComponent`로 등록.
- `new ComponentContext(surface, componentId)`로 컨텍스트 생성 후 반환.

기본값: `componentId = 'test-comp'`, `componentType = 'TestComponent'`, `data = {}`.

---

### `should be created`

`ComponentBinder` 인스턴스가 truthy임을 확인.

---

### `basic property binding` — `should bind literal properties to Angular signals`

**검증 동작**: `string`, `number`, `boolean`, `object`, `null` 등 다양한 리터럴 값이 `bound[key].value()`로 올바르게 읽히는지 확인. `template`이 `undefined`이고 `raw`가 원본 값임을 확인.

---

### `basic property binding` — `should bind path-based properties and setup onUpdate (two-way binding)`

**검증 동작**: `{ path: '/data/text' }` 형태의 프로퍼티가 DataModel에서 초기값 `'initial-value'`를 읽고, `onUpdate('new-value')` 호출 후 `surface.dataModel.get('/data/text')`가 업데이트되며 signal 값도 반영됨을 확인.

---

### `basic property binding` — `should make onUpdate a no-op for literal properties`

**검증 동작**: 리터럴 프로퍼티에 대해 `onUpdate` 호출이 예외 없이 실행되고 값이 변경되지 않음을 확인.

---

### `single child bindings (child, trigger, content)` — 각 키별 3가지 케이스

`testSingleChildKey(key)` 헬퍼로 `'child'`, `'trigger'`, `'content'` 각각에 대해 반복 실행:

1. **null/falsy 처리**: 값이 null이면 `bound[key].value()`가 null임을 확인.
2. **문자열 ID → Child 객체 변환**: `'my-child-id'`가 `{ id: 'my-child-id', basePath: '/' }`로 변환됨을 확인.
3. **기존 Child 객체 유지**: `{ id: 'custom-child-id', basePath: '/different/path' }` 형태 객체가 그대로 반환됨을 확인.

---

### `children list bindings` — 5가지 케이스

1. **null/falsy 처리**: `children: null` → `value()`가 빈 배열 반환.
2. **정적 문자열 배열**: `['child-1', 'child-2']` → `[{ id: 'child-1', basePath: '/' }, { id: 'child-2', basePath: '/' }]` 반환. `template`은 undefined.
3. **path 기반 동적 배열 (문자열 ID 목록)**: `{ path: '/dynamic/list' }` + 데이터 `['child-a', 'child-b']` → Child 객체 배열. `template`은 undefined.
4. **path 기반 동적 배열 (기존 Child 객체 목록)**: 데이터가 이미 `{ id, basePath }` 형태이면 그대로 반환.
5. **ChildList 템플릿 확장**: `{ componentId: 'item-card', path: '/items' }` + 데이터 `['item1', 'item2', 'item3']` → 인덱스 기반 basePath(`'/items/0'`, `'/items/1'`, `'/items/2'`)를 가진 Child 배열. `template`이 `{ id: 'item-card', path: '/items' }`로 설정됨.
6. **template 비누출**: `children`의 template이 다른 프로퍼티(`anotherProp`)의 `template`에 영향을 주지 않음.

---

### `validation checks (checks)` — 4가지 케이스

1. **null/빈 배열**: `isValid.value()` = true, `validationErrors.value()` = [].
2. **조건 기반 룰**: `{ condition: { path: '...' }, message: '...' }` 형태의 두 룰 중 하나가 false이면 `isValid` = false, `validationErrors`에 해당 메시지 포함. DataModel 업데이트 시 reactive하게 재평가.
3. **shorthand 룰**: `{ path: '/form/singleCheck' }` 형태 룰(condition이 곧 룰 자체). 기본 메시지 `'Validation failed'` 사용.
4. **다중 오류 동적 평가**: 3개 룰 중 2개 false → 오류 2개. 부분 수정 → 오류 1개. 전체 수정 → 오류 없음. DataModel 변경에 따라 reactive하게 반응.

## 동작 흐름

각 테스트는 `createComponentContext`로 실제 A2UI 모델 객체를 구성하고, `binder.bind(context)` 결과를 assertion한다. 반응성 테스트는 `surface.dataModel.set(path, value)` 호출 후 signal `.value()`를 재평가하여 동적 업데이트를 검증한다.
