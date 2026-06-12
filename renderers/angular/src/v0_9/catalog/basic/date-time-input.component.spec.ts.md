# renderers/angular/src/v0_9/catalog/basic/date-time-input.component.spec.ts

## 개요

`DateTimeInputComponent`에 대한 Angular 단위 테스트 파일이다. Jasmine/TestBed를 사용해 컴포넌트 생성, 날짜 입력 렌더링, 값 변경 시 `onUpdate` 호출, 빈 값 처리 등 핵심 동작을 검증한다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture`, `TestBed`
- `@angular/core` — `signal as angularSignal`

### 저장소 내부 모듈
- [`./date-time-input.component`](./date-time-input.component.ts.md) — `DateTimeInputComponent`
- [`../../core/a2ui-renderer.service`](../../core/a2ui-renderer.service.ts.md) — `A2uiRendererService`
- [`../../core/component-binder.service`](../../core/component-binder.service.ts.md) — `ComponentBinder`
- [`../../core/test-utils`](../../core/test-utils.ts.md) — `setComponentProps`, `createBoundProperty`, `ComponentToProps`

## Exports

없음 (테스트 파일).

## 테스트 케이스 명세

### 픽스처 및 모킹 (`beforeEach` — async)

- `mockRendererService`: `surfaceGroup.getSurface`를 Jasmine 스파이로 모킹. 반환값은 빈 `componentsModel` Map과 `id: 'mock-catalog'`, 빈 `components` Map을 가진 `catalog` 객체.
- `mockBinder`: `ComponentBinder`의 `bind` 메서드를 Jasmine 스파이로 모킹(`createSpyObj`).
- `TestBed.configureTestingModule`에 `DateTimeInputComponent`를 imports, 위 두 mock을 provider로 등록 후 `compileComponents()`.
- `TestBed.createComponent(DateTimeInputComponent)`로 fixture 생성.
- `fixture.componentRef.setInput('surfaceId', 'test-surface')`, `setInput('dataContextPath', '/')` 설정.
- `defaultProps`: `label: createBoundProperty<string | undefined>('')`, `value: createBoundProperty('')`, `enableDate: createBoundProperty<boolean | undefined>(true)`, `enableTime: createBoundProperty<boolean | undefined>(true)`, `isValid: createBoundProperty(true)`, `validationErrors: createBoundProperty<string[]>([])`.
- `setComponentProps(fixture, defaultProps)` 호출.

### `should create`

- `fixture.detectChanges()` 실행 후 `component`가 truthy인지 `expect(component).toBeTruthy()` 확인.
- 기본 프롭으로 컴포넌트가 오류 없이 생성되는지 검증.

### `should render date input`

- 프롭을 재설정: `label: createBoundProperty<string | undefined>('Start Date')`, `value: createBoundProperty('2026-03-16')`, `enableTime: createBoundProperty<boolean | undefined>(false)`.
- `fixture.detectChanges()` 후 `fixture.nativeElement.querySelector('input[type="date"]')`로 날짜 입력 엘리먼트를 조회.
- 엘리먼트가 truthy인지, `input.value`가 `'2026-03-16'`인지 검증.
- `enableTime: false`이므로 시간 입력은 렌더링되지 않고 날짜 입력만 존재함을 확인.

### `should call onUpdate when date or time changes`

- `value` 프롭을 `angularSignal('2026-03-16T10:00:00')`과 `onUpdate: onUpdateSpy`를 직접 조합해 설정.
- `fixture.detectChanges()` 후 `input[type="date"]`와 `input[type="time"]` 엘리먼트를 각각 조회.
- 날짜 입력값을 `'2026-03-17'`로 변경하고 `dispatchEvent(new Event('change'))` 발생 → `onUpdateSpy`가 `'2026-03-17T10:00:00'`으로 호출되었는지 검증. (기존 시간 `'10:00:00'` 보존)
- `onUpdateSpy.calls.reset()` 후 시간 입력값을 `'11:00'`으로 변경하고 `change` 이벤트 발생 → `onUpdateSpy`가 `'2026-03-16T11:00:00'`으로 호출되었는지 검증. (기존 날짜 `'2026-03-16'` 보존, 초 `:00` 자동 보완)

### `should handle empty value by returning empty strings`

- `value: createBoundProperty('')`로 설정하고 `fixture.detectChanges()`.
- `input[type="date"]`의 `value`가 `''`인지, `input[type="time"]`의 `value`가 `''`인지 검증.
- 빈 값 입력 시 날짜·시간 파싱이 빈 문자열을 반환하고 오류가 발생하지 않음을 확인.

## 동작 흐름

각 테스트는 단일 `beforeEach` 블록으로 픽스처를 초기화한다. `setComponentProps`로 프롭을 설정하고 `fixture.detectChanges()`로 변경 감지를 트리거한 후, DOM 쿼리 또는 스파이 호출 여부로 동작을 검증한다.
