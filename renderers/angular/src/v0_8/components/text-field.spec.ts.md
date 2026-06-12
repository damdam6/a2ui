# renderers/angular/src/v0_8/components/text-field.spec.ts

## 개요

`TextField` Angular 컴포넌트의 단위 테스트 파일이다. `TestBed`를 사용하여 컴포넌트를 격리된 환경에서 마운트하고, 레이블 렌더링, 입력 타입 전환, 사용자 입력 이벤트 처리 등 핵심 동작을 검증한다. `MessageProcessor`와 `Theme`, `Catalog`를 모의(mock) 객체로 주입하여 외부 의존성을 차단한다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture`, `TestBed`
- `@angular/platform-browser` — `By`

### 저장소 내부 모듈
- [`./text-field`](./text-field.ts.md) — 테스트 대상 컴포넌트 `TextField`
- [`../data/processor`](../data/processor.ts.md) — `MessageProcessor` (mock)
- [`../rendering/theming`](../rendering/theming.ts.md) — `Theme` (mock)
- [`../rendering/catalog`](../rendering/catalog.ts.md) — `Catalog` (mock)

## Exports

없음 (테스트 파일).

## 테스트 케이스 명세

### 픽스처 및 모킹 설정 (`beforeEach`)

`TestBed`에 다음을 설정한다:
- `mockTheme`: `new Theme()` 인스턴스. `components.TextField`를 `{container: 'tf-container', label: 'tf-label', element: 'tf-input'}`으로 직접 할당.
- `mockProcessor`: `jasmine.createSpyObj`로 생성. `dispatch`, `resolvePath`, `getData` 메서드를 스파이로 등록. `dispatch`는 `Promise.resolve([])` 반환.
- `TestBed.configureTestingModule`에 `TextField`를 `imports`로, 세 mock을 `providers`로 등록.
- 컴포넌트 입력값: `surfaceId='surface-1'`, `component={id: 'tf-1', type: 'TextField', weight: 1}`, `weight=1`, `label={literalString: 'Name'}`, `text={literalString: 'John Doe'}`.

---

### `should create`

컴포넌트 인스턴스가 정상적으로 생성되는지 확인한다(`expect(component).toBeTruthy()`).

---

### `should render label and set input attributes`

- `By.css('label')`로 `<label>` 요소를 조회하고, `textContent.trim()`이 `'Name'`인지 검증.
- `By.css('input')`로 `<input>` 요소를 조회하고, `value`가 `'John Doe'`, `type`이 `'text'`(기본값)인지 검증.

---

### `should change input type based on textFieldType`

- `textFieldType` 입력을 `'number'`로 변경 후 `detectChanges()`, `<input>` 타입이 `'number'`인지 확인.
- 이어서 `'date'`로 변경 후, 타입이 `'date'`인지 확인.

---

### `should trigger sendAction on input update`

- `<input>` 요소의 `value`를 `'Jane Doe'`로 변경하고 `new Event('input')`을 디스패치.
- `mockProcessor.dispatch`가 호출되었는지 확인.
- 전달된 메시지의 `userAction.name`이 `'input'`이고 `userAction.context`가 `{value: 'Jane Doe'}`인지 검증.

## 동작 흐름

`beforeEach`에서 테스트 환경을 초기화하고 컴포넌트를 마운트한 뒤, 각 테스트가 독립적으로 특정 시나리오를 검증한다. 입력 타입 변경 테스트는 Angular의 신호(signal) 기반 반응성을, 액션 트리거 테스트는 `MessageProcessor.dispatch` 위임 경로를 확인한다.
