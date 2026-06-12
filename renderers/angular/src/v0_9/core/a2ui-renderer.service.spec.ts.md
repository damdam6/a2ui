# renderers/angular/src/v0_9/core/a2ui-renderer.service.spec.ts

## 개요

`A2uiRendererService`의 단위 테스트 파일이다. `A2UI_RENDERER_CONFIG` 주입 토큰을 통해 목(mock) 카탈로그를 제공하고, 서비스 생성, `surfaceGroup` 초기화, `processMessages` 위임, `ngOnDestroy` 시 dispose 호출을 검증한다.

## 의존성

### 외부 패키지
- `@angular/core/testing`: `TestBed`

### 저장소 내부 모듈
- [`./a2ui-renderer.service`](./a2ui-renderer.service.ts.md) — `A2uiRendererService`, `A2UI_RENDERER_CONFIG`

## Exports

없음 (테스트 파일).

## 테스트 케이스 명세

### 픽스처 및 모킹 설정

**`beforeEach`**: 각 테스트 전에 `mockCatalog`를 직접 객체 리터럴로 구성한다. `mockCatalog`는 다음 구조를 가진다:
- `components`: `new Map()`
- `functions`: `new Map()`
- `get invoker()`: 함수명으로 `functions` 맵에서 조회하여 호출하는 getter. 함수가 없으면 `console.warn`을 출력하고 `undefined` 반환.

`TestBed.configureTestingModule`에 `A2uiRendererService`와 `A2UI_RENDERER_CONFIG: { catalogs: [mockCatalog] }`를 등록하고, `TestBed.inject(A2uiRendererService)`로 서비스 인스턴스를 획득한다.

---

### `should be created`

**검증 동작**: 서비스 인스턴스가 truthy임을 확인한다.

---

### `initialization` — `should create surfaceGroup`

**검증 동작**: `service.surfaceGroup`이 정의되어 있음(not undefined)을 확인한다.

---

### `processMessages` — `should delegate to MessageProcessor`

**검증 동작**: `service.processMessages([])`를 호출했을 때 예외가 발생하지 않음을 확인한다. 빈 배열을 전달하여 내부 `MessageProcessor` 위임 경로가 오류 없이 실행됨을 간접적으로 검증한다.

---

### `ngOnDestroy` — `should dispose surfaceGroup`

**검증 동작**: `service.ngOnDestroy()` 호출 시 `surfaceGroup`의 `dispose` 메서드가 호출됨을 확인한다.  
**픽스처/모킹**: `service.surfaceGroup`에 대해 `spyOn(surfaceGroup as any, 'dispose')`를 설정한 뒤 `ngOnDestroy()`를 호출하고 `disposeSpy`가 호출되었는지 검사한다.

## 동작 흐름

테스트는 `TestBed`를 통해 실제 `A2uiRendererService`를 인스턴스화하되, 카탈로그만 가벼운 목 객체로 교체한다. 각 테스트 그룹은 서비스의 주요 공개 인터페이스(`surfaceGroup`, `processMessages`, `ngOnDestroy`)를 독립적으로 검증한다.
