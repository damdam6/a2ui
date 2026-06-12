# renderers/angular/src/v0_8/components/checkbox.spec.ts

## 개요

`Checkbox` Angular 컴포넌트에 대한 단위 테스트 파일이다. 레이블 렌더링, 체크 상태 반영, 체크박스 토글 시 `toggle` 액션 디스패치 동작을 검증한다. `Catalog` 프로바이더 없이 구성되며 `MessageProcessor` 스파이를 통해 디스패치 호출을 추적한다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture`, `TestBed`

### 저장소 내부 모듈
- [`./checkbox`](./checkbox.ts.md) — `Checkbox` (테스트 대상)
- [`../data/processor`](../data/processor.ts.md) — `MessageProcessor`
- [`../rendering/theming`](../rendering/theming.ts.md) — `Theme`
- `../types` (타입 전용) — `A2UIClientEventMessage`, `CheckboxNode`

## 테스트 케이스

### 픽스처 및 모킹 설정 (`beforeEach`)
- `mockProcessor`: `dispatch`, `resolvePath`, `getData` 메서드를 가진 스파이 객체. `dispatch`는 `Promise.resolve([])`를 반환하도록 설정.
- `mockNode`: `id: 'chk-1'`, `type: 'CheckBox'`, `weight: 1`, `properties.label.literalString: 'Accept Terms'`, `properties.value.literalBoolean: false`를 가진 `CheckboxNode` 픽스처.
- 입력: `surfaceId: 'surface-1'`, `component: mockNode`, `weight: 1`, `label: {literalString: 'Accept Terms'}`, `value: {literalBoolean: false}`.
- `Catalog` 프로바이더는 등록하지 않는다 (체크박스가 자식 컴포넌트를 렌더링하지 않으므로 불필요).

### 테스트: `'should create'`
- 검증: `component`가 truthy임을 확인.

### 테스트: `'should render label text'`
- 검증: `<label>` 요소의 `textContent`가 `'Accept Terms'`를 포함하는지 확인.
- 사용 픽스처: `label: {literalString: 'Accept Terms'}` 입력.

### 테스트: `'should reflect checked state'`
- 검증 1: 초기 상태에서 `<input>` 요소의 `checked`가 `false`임을 확인.
- 동작: `value` 입력을 `{literalBoolean: true}`로 변경하고 `detectChanges()`.
- 검증 2: `<input>`의 `checked`가 `true`로 변경되었음을 확인.
- 경계 케이스: 신호 변경이 DOM에 즉시 반영되는지 검증.

### 테스트: `'should dispatch action on toggle'`
- 동작: `<input>`의 `checked`를 `true`로 수동 설정하고 `new Event('change')`를 디스패치한다.
- 검증: `mockProcessor.dispatch`가 호출되었는지, 최근 호출 인수의 `userAction.name`이 `'toggle'`이고 `userAction.context['checked']`가 `true`인지 확인.
- 사용 모킹: `mockProcessor.dispatch` 스파이로 전달된 `A2UIClientEventMessage` 구조 검증.

## 동작 흐름

`beforeEach`에서 `TestBed` 환경을 구성하고 리터럴 값을 입력으로 설정한다. 체크 상태 변경 테스트는 `setInput`으로 입력을 동적으로 변경하고 `detectChanges()`를 트리거한다. 토글 액션 테스트는 네이티브 DOM 이벤트를 직접 발생시켜 `onToggle` 핸들러를 실행하고 `dispatch` 호출을 확인한다.
