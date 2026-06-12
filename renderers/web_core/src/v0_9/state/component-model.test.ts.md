# renderers/web_core/src/v0_9/state/component-model.test.ts

## 개요

`ComponentModel` 클래스의 기본 동작을 `node:test`와 `node:assert`를 사용해 검증하는 테스트 파일이다. 초기화, 프로퍼티 업데이트, 업데이트 알림, 구독 해제, 컴포넌트 트리 표현 반환 등 핵심 동작을 커버한다.

## 의존성

### 외부 패키지
- `node:assert`
- `node:test` (`describe`, `it`, `beforeEach`)

### 저장소 내부 모듈
- [`./component-model.js`](./component-model.ts.md) — `ComponentModel`

## Exports

없음. 테스트 파일로 실행 전용이다.

## 테스트 케이스 명세

모든 테스트는 `describe('ComponentModel', ...)` 블록 내에 있다.

**픽스처/beforeEach**: 각 테스트 전에 `new ComponentModel('c1', 'Button', {label: 'Click Me'})`로 새 인스턴스를 생성하여 `component` 변수에 저장한다.

---

### `it('initializes properties')`

**검증하는 동작**: 생성자에 전달한 `id`, `type`, `properties`가 올바르게 저장되는지 확인한다.

**검증 방법**:
- `assert.strictEqual(component.id, 'c1')`
- `assert.strictEqual(component.type, 'Button')`
- `assert.strictEqual(component.properties.label, 'Click Me')`

---

### `it('updates properties')`

**검증하는 동작**: `component.properties`에 새 객체를 할당하면 해당 값으로 프로퍼티가 교체되는지 확인한다.

**검증 방법**: `component.properties = {label: 'Clicked'}` 실행 후 `assert.strictEqual(component.properties.label, 'Clicked')`.

---

### `it('notifies listeners on update')`

**검증하는 동작**: `onUpdated.subscribe(listener)`로 등록한 리스너가 `properties` 할당 시 해당 `ComponentModel` 인스턴스를 인수로 받아 호출되는지 확인한다.

**검증 방법**:
1. `let updatedComponent: ComponentModel | undefined`를 선언한다.
2. `component.onUpdated.subscribe((c) => { updatedComponent = c; })`로 구독한다.
3. `component.properties = {label: 'New'}` 할당.
4. `assert.strictEqual(updatedComponent, component)` — 동일 인스턴스가 전달됨을 확인.
5. `assert.strictEqual(updatedComponent?.properties.label, 'New')` — 업데이트된 값도 확인.

---

### `it('unsubscribes listeners')`

**검증하는 동작**: `sub.unsubscribe()` 호출 이후에는 리스너가 더 이상 호출되지 않는지 확인한다.

**검증 방법**:
1. `let callCount = 0` 선언.
2. `const sub = component.onUpdated.subscribe(() => { callCount++; })`로 구독.
3. `component.properties = {label: '1'}` → `assert.strictEqual(callCount, 1)`.
4. `sub.unsubscribe()` 호출.
5. `component.properties = {label: '2'}` → `assert.strictEqual(callCount, 1)` — 증가하지 않음을 확인.

---

### `it('returns component tree representation')`

**검증하는 동작**: `component.componentTree`가 `id`, `type`, 그리고 `properties`의 키들을 flat하게 포함하는 객체를 반환하는지 확인한다.

**검증 방법**: `assert.deepStrictEqual(component.componentTree, { id: 'c1', type: 'Button', label: 'Click Me' })`.

## 동작 흐름

`beforeEach`가 각 테스트 전에 새 `ComponentModel` 인스턴스를 생성하므로 테스트 간 상태가 격리된다. 모킹 없이 실제 `ComponentModel` 인스턴스를 직접 사용한다.
