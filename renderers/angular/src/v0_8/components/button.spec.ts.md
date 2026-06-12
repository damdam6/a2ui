# renderers/angular/src/v0_8/components/button.spec.ts

## 개요

`Button` Angular 컴포넌트에 대한 단위 테스트 파일이다. `Renderer` 디렉티브를 `MockRenderer`로 교체하여 트리 렌더링 없이 `Button` 컴포넌트를 격리 테스트한다. 테마 스타일 적용, 클릭 시 액션 디스패치, `action`이 `null`일 때의 비디스패치 동작을 검증한다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture`, `TestBed`
- `@angular/core` — `Directive`, `Input`

### 저장소 내부 모듈
- [`./button`](./button.ts.md) — `Button` (테스트 대상)
- [`../data/processor`](../data/processor.ts.md) — `MessageProcessor`
- [`../rendering/theming`](../rendering/theming.ts.md) — `Theme`
- [`../rendering/catalog`](../rendering/catalog.ts.md) — `Catalog`
- [`../rendering/renderer`](../rendering/renderer.ts.md) — `Renderer` (교체 대상)
- `../types` (타입 전용) — `Action`, `ButtonNode`, `A2UIClientEventMessage`

## 테스트 케이스

### 내부 클래스: `MockRenderer`
- `@Directive({ selector: '[a2ui-renderer]', standalone: true })`로 선언된 테스트 전용 가짜 디렉티브.
- `@Input() surfaceId: string`, `@Input() component: any`를 가진다.
- 실제 `Renderer` 대신 사용되어 자식 컴포넌트 트리 렌더링 없이 격리 테스트를 가능하게 한다.

### 픽스처 및 모킹 설정 (`beforeEach`)
- `mockProcessor`: `dispatch`, `resolvePath`, `getData` 메서드를 가진 스파이 객체. `dispatch`는 `Promise.resolve([])`를 반환하도록 설정.
- `mockTheme`: `components.Button: 'btn-class'`, `additionalStyles.Button: {color: 'red'}`로 설정한 실제 `Theme` 인스턴스.
- `mockAction`: `name: 'testAction'`, `context: []`를 가진 `Action` 픽스처.
- `mockButtonNode`: `id: 'btn-1'`, `type: 'Button'`, `weight: 1`, `properties.child: {id: 'text-1', type: 'Text', properties: {text: 'Click Me'}}`, `properties.action: mockAction`을 가진 `ButtonNode` 픽스처.
- `TestBed.overrideComponent`에서 `Button`의 `imports`에서 실제 `Renderer`를 제거하고 `MockRenderer`를 추가한다.
- 입력: `surfaceId: 'surface-1'`, `component: mockButtonNode`, `weight: 1`, `action: mockAction`.

### 테스트: `'should create'`
- 검증: `component`가 truthy임을 확인.

### 테스트: `'should apply theme classes and styles'`
- 검증: `<button>` 요소의 `className`이 `'btn-class'`를 포함하고, `style.color`가 `'red'`임을 확인.
- 사용 모킹: `mockTheme.components.Button`, `mockTheme.additionalStyles.Button`.

### 테스트: `'should call super.sendAction on click'`
- 검증: 버튼 클릭 후 `mockProcessor.dispatch`가 호출되었는지, 디스패치된 메시지의 `userAction.name`이 `'testAction'`이고 `userAction.sourceComponentId`가 `'btn-1'`인지 확인.
- 사용 픽스처: `mockButtonNode`, `mockAction`.

### 테스트: `'should not dispatch if action is null'`
- 검증: `action` 입력을 `null`로 설정한 후 버튼 클릭 시 `mockProcessor.dispatch`가 호출되지 않음을 확인.
- 경계 케이스: `action`이 `null`일 때 `handleClick` 내 조기 반환 분기를 검증.

## 동작 흐름

`beforeEach`에서 `TestBed`를 구성하고 `overrideComponent`로 실제 `Renderer`를 `MockRenderer`로 교체한다. 각 테스트는 독립적으로 입력을 설정하고 `detectChanges()`를 트리거한 후 DOM 조작(`.click()`, `setInput`)을 통해 동작을 유도하고 결과를 단언한다.
