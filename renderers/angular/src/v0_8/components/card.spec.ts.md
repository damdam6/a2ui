# renderers/angular/src/v0_8/components/card.spec.ts

## 개요

`Card` Angular 컴포넌트에 대한 단위 테스트 파일이다. `Renderer` 디렉티브를 인스턴스 추적 기능이 있는 `MockRenderer`로 교체하여 자식 렌더링 횟수를 검증한다. 테마 클래스 적용, 단일 자식 렌더링, 복수 자식 렌더링 동작을 테스트한다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture`, `TestBed`
- `@angular/core` — `Directive`, `Input`

### 저장소 내부 모듈
- [`./card`](./card.ts.md) — `Card` (테스트 대상)
- [`../data/processor`](../data/processor.ts.md) — `MessageProcessor`
- [`../rendering/theming`](../rendering/theming.ts.md) — `Theme`
- [`../rendering/catalog`](../rendering/catalog.ts.md) — `Catalog`
- `../types` (타입 전용) — `CardNode`

## 테스트 케이스

### 내부 클래스: `MockRenderer`
- `@Directive({ selector: '[a2ui-renderer]', standalone: true })`로 선언된 테스트 전용 가짜 디렉티브.
- `@Input() surfaceId: string`, `@Input() component: any`를 가진다.
- `static instances: MockRenderer[]` 배열을 통해 생성된 인스턴스 수를 추적한다.
- 생성자에서 `MockRenderer.instances.push(this)`를 실행하여 인스턴스를 등록한다.

### 픽스처 및 모킹 설정 (`beforeEach`)
- `mockTheme`: `components.Card: 'card-class'`로 설정한 실제 `Theme` 인스턴스.
- `mockNode`: `id: 'card-1'`, `type: 'Card'`, `weight: 1`, `properties.child: {id: 'dummy-1', type: 'Text', ...}`, `properties.children: []`를 가진 `CardNode` 픽스처.
- `TestBed.overrideComponent`의 `set.imports`를 `[MockRenderer]`로 지정하여 `Renderer`를 완전 교체한다.
- `beforeEach` 끝에서 `MockRenderer.instances = []`로 인스턴스 추적 배열을 초기화한다.
- 입력: `surfaceId: 'surface-1'`, `component: mockNode`, `weight: 1`.

### 테스트: `'should create'`
- 검증: `component`가 truthy임을 확인.

### 테스트: `'should apply theme class'`
- 검증: `<div>` 요소의 `className`이 `'card-class'`를 포함하는지 확인.

### 테스트: `'should render child if provided'`
- 동작: `child` 입력을 `{id: 'child-1', type: 'Text', properties: {}}`로 설정하고 `detectChanges()`.
- 검증: `MockRenderer.instances.length`가 `1`임을 확인하여 단일 자식이 렌더링되었음을 검증한다. DOM 쿼리 대신 인스턴스 추적을 사용한다. 디버그용 `console.log`가 포함되어 있다.

### 테스트: `'should render children if provided'`
- 동작: `children` 입력을 `[{id: 'child-1', ...}, {id: 'child-2', ...}]`로 설정하고 `detectChanges()`.
- 검증: `MockRenderer.instances.length`가 `2`임을 확인하여 두 자식이 모두 렌더링되었음을 검증한다.

## 동작 흐름

`beforeEach`에서 `TestBed`를 구성하고 `overrideComponent`로 `Renderer`를 `MockRenderer`로 교체한다. 인스턴스 추적 배열을 `beforeEach` 종료 시마다 초기화하여 테스트 간 오염을 방지한다. 각 테스트는 `setInput`으로 입력을 동적으로 변경하고 `detectChanges()` 후 `MockRenderer.instances.length`를 확인한다.
