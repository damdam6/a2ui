# renderers/angular/src/v0_8/data/processor.spec.ts

## 개요

`MessageProcessor` Angular 서비스의 단위 테스트 파일이다. 메시지 처리, 이벤트 디스패치 및 완료 해결, 데이터 접근/설정, 경로 해석, 표면(surface) 관리 등 `MessageProcessor`의 모든 공개 메서드를 검증한다. 내부 `baseProcessor`를 스파이로 감시하여 위임 동작을 확인한다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `TestBed`
- `@a2ui/web_core/v0_8` — `Surface` 타입 (as `WebCore`)

### 저장소 내부 모듈
- [`./processor`](./processor.ts.md) — `MessageProcessor`, `A2UIClientEvent`
- [`../rendering/catalog`](../rendering/catalog.ts.md) — `Catalog` (mock)
- [`../types`](../types.ts.md) — `A2UIClientEventMessage`, `AnyComponentNode`, `ServerToClientMessage` (타입)

## Exports

없음 (테스트 파일).

## 테스트 케이스 명세

### 픽스처 설정 (`beforeEach`)

`TestBed.configureTestingModule`에 `MessageProcessor`와 `{provide: Catalog, useValue: {}}`를 등록하고, `TestBed.inject(MessageProcessor)`로 서비스 인스턴스를 획득한다.

---

### `should be created`

서비스 인스턴스가 정상적으로 생성되는지 확인한다.

---

### `should forward processMessages to baseProcessor`

`(service as any).baseProcessor.processMessages`를 스파이 등록. `service.processMessages([])`를 호출하고 스파이가 같은 인자로 호출되었는지 확인한다.

---

### `should dispatch events and emit to observable`

`service.events.subscribe`로 구독 후 `service.dispatch(mockMessage)`를 호출. 콜백에서 수신된 `event.message`가 `mockMessage`와 동일 참조인지, `event.completion`이 truthy인지 검증. `done()` 패턴으로 비동기 완료 처리.

---

### `should resolve dispatch promise when completion is triggered`

`service.events.subscribe`에서 수신된 `event.completion.next(replyMessages)`를 즉시 호출. `await service.dispatch(mockMessage)`의 결과가 `replyMessages`와 동일 참조인지 검증하여, `completion` Subject가 Promise를 해결하는 경로를 확인한다.

---

### `should forward getData to baseProcessor`

`baseProcessor.getData`를 스파이로 등록하고 `'mock-value'` 반환 설정. `service.getData(node, 'path/to/data', 'surf-1')` 호출 후 스파이 호출 인자와 반환값을 검증.

---

### `should forward setData to baseProcessor`

`baseProcessor.setData`를 스파이로 등록. `service.setData(node, 'path/to/data', 'new-value', 'surf-1')` 호출 후 스파이가 같은 인자로 호출되었는지 확인.

---

### `should forward resolvePath to baseProcessor`

`baseProcessor.resolvePath`를 스파이로 등록하고 `'resolved-path'` 반환 설정. `service.resolvePath('path/to/data', 'context')` 결과가 `'resolved-path'`인지 확인.

---

### `should clear surfaces`

`baseProcessor.clearSurfaces`를 스파이로 등록. `service.clearSurfaces()` 호출 후 스파이가 실행되었는지 확인.

---

### `should only return surfaces that are ready to render`

- `readySurface`: `rootComponentId`가 `'ready-component-id'`이고 `componentTree`가 Text 노드로 설정된 `WebCore.Surface`.
- `notReadySurface`: `rootComponentId`와 `componentTree`가 모두 `null`.
- `baseProcessor.surfaces`에 두 surface를 직접 Map으로 설정.
- `service.getSurfaces()` 결과가 크기 1이고, `readySurface`만 포함하며 `notReadySurface`는 제외됨을 검증.

## 동작 흐름

각 테스트는 내부 `baseProcessor` 인스턴스의 메서드를 Jasmine 스파이로 교체하거나 상태를 직접 조작하여, `MessageProcessor`가 올바르게 위임하고 변환하는지 화이트박스 방식으로 검증한다.
