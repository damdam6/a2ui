# renderers/web_core/src/v0_9/state/surface-components-model.test.ts

## 개요

`SurfaceComponentsModel` 클래스의 동작을 Node.js 내장 테스트 러너(`node:test`)로 검증하는 테스트 파일이다. 컴포넌트 추가·조회·업데이트·삭제, 이벤트 전파, 에러 처리, dispose 흐름을 커버한다.

## 의존성

### 저장소 내부 모듈

- [`./surface-components-model.ts`](./surface-components-model.ts.md) — `SurfaceComponentsModel` 임포트
- [`./component-model.ts`](./component-model.ts.md) — `ComponentModel` 임포트

### 외부 패키지

- `node:assert` — `assert`
- `node:test` — `describe`, `it`, `beforeEach`

## Exports

없음 (테스트 엔트리포인트)

## 테스트 케이스 명세

### 픽스처 (`beforeEach`)

매 테스트 전 `new SurfaceComponentsModel()`으로 빈 모델을 생성한다.

---

### 케이스 목록

| 케이스명 | 검증 동작 | 픽스처/모킹 |
|---|---|---|
| `starts empty` | `model.get('any')` → `undefined` | 없음 |
| `adds a new component` | `ComponentModel('c1', 'Button', {label: 'Click'})` 추가 후 `get('c1')`로 id·type·properties 확인 | `ComponentModel` 인스턴스 직접 생성 |
| `updates an existing component` | `c1.properties = {label: 'Updated'}` 설정 시 `onUpdated` 이벤트 1회 발생 및 값 갱신 확인 | `c1.onUpdated.subscribe()`로 카운터 감시 |
| `notifies on component creation` | `addComponent()` 호출 후 `onCreated` 구독자에게 `ComponentModel` 전달 및 ID 확인 | `model.onCreated.subscribe()` |
| `throws when adding duplicate component` | 동일 ID로 두 번 `addComponent()` 시 `/already exists/` 에러 | `assert.throws` |
| `returns entries iterator` | `c1`, `c2` 추가 후 `Array.from(model.entries)` → 길이 2, 순서 일치 확인 | `ComponentModel` 2개 |
| `disposes components during model dispose` | `model.dispose()` 후 `c1.dispose`가 호출되었는지 및 내부 `components.size === 0` 확인 | `c1.dispose`를 모킹하여 `childDisposed` 플래그 설정, `(model as any).components.size` 직접 접근 |
| `safely attempts to remove non-existent component` | `model.removeComponent('does-not-exist')` 시 에러 없음 | 없음 |
