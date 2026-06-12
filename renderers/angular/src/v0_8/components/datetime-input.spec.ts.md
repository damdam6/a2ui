# renderers/angular/src/v0_8/components/datetime-input.spec.ts

## 개요

`DateTimeInput` 컴포넌트에 대한 Angular 단위 테스트 파일이다. `TestBed`를 사용해 `DateTimeInput`을 격리된 환경에서 마운트하고, 렌더링·입력 타입 결정·이벤트 처리·테마 클래스 적용을 검증한다.

## 의존성

### 외부 패키지
- `@angular/core/testing`: `ComponentFixture`, `TestBed`

### 저장소 내부 모듈
- [`./datetime-input`](./datetime-input.ts.md): 테스트 대상 `DateTimeInput` 컴포넌트
- [`../types`](../types.ts.md): `A2UIClientEventMessage`, `DateTimeInputNode` 타입
- [`../rendering/theming`](../rendering/theming.ts.md): `Theme` 서비스
- [`../data/processor`](../data/processor.ts.md): `MessageProcessor` 서비스
- [`../rendering/catalog`](../rendering/catalog.ts.md): `Catalog` 서비스

## Exports

없음 (테스트 파일).

## 테스트 케이스 명세

### 픽스처 / 모킹 설정 (`beforeEach`)

- `mockProcessor`: `jasmine.createSpyObj`로 생성. `dispatch` 메서드가 `Promise.resolve([])`를 반환하도록 스텁 처리.
- `mockTheme`: `new Theme()`로 생성 후 `components.DateTimeInput`을 `{container: {'dt-container': true}, label: {'dt-label': true}, element: {'dt-element': true}}`로 설정.
- `mockDatetimeNode`: `id: 'dt-1'`, `type: 'DateTimeInput'`, `weight: 1`, `properties.value: {literalString: '2023-10-27'}`.
- `TestBed` 제공자: `MessageProcessor → mockProcessor`, `Theme → mockTheme`, `Catalog → {}`.
- 입력값 설정: `surfaceId: 'surface-1'`, `component: mockDatetimeNode`, `weight: 1`, `value: mockDatetimeNode.properties.value`, `label: {literalString: 'Select Date'}`.

### 테스트 케이스

| 케이스명 | 검증 동작 | 픽스처/모킹 |
|----------|-----------|-------------|
| `should create` | 컴포넌트 인스턴스가 truthy인지 확인 | 기본 설정 |
| `should render label and input element` | DOM에서 `<label>`과 `<input>` 요소가 존재하는지, 레이블 텍스트가 `'Select Date'`를 포함하는지, 입력값이 `'2023-10-27'`인지 확인 | 기본 설정 |
| `should apply correct input type based on flags` | 기본 상태(`enableDate=true`, `enableTime=false`)에서 `input.type`이 `'date'`, `enableDate=false`·`enableTime=true`이면 `'time'`, 둘 다 true이면 `'datetime-local'`인지 세 단계로 검증 | `setInput`으로 `enableDate`, `enableTime` 동적 변경 |
| `should call super.sendAction on change event` | `<input>`에서 `'change'` 이벤트 디스패치 후 `mockProcessor.dispatch`가 호출되었는지, 전달된 메시지의 `userAction.name`이 `'change'`이고 `userAction.context`가 `{value: '2023-10-28'}`인지 확인 | `mockProcessor.dispatch` spy |
| `should apply theme classes` | `<div>`, `<label>`, `<input>` 각각에 `'dt-container'`, `'dt-label'`, `'dt-element'` 클래스가 적용되어 있는지 확인 | `mockTheme.components.DateTimeInput` |

## 동작 흐름

각 테스트는 `TestBed`를 통해 `DateTimeInput`을 컴파일하고, `setInput`으로 필수 입력을 설정한 뒤 `detectChanges`를 호출해 뷰를 업데이트한다. DOM 쿼리(`querySelector`, `debugElement.query`) 또는 spy 호출 내역으로 기대 동작을 검증한다.
