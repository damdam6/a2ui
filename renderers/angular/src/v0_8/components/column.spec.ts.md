# renderers/angular/src/v0_8/components/column.spec.ts

## 개요

`Column` Angular 컴포넌트에 대한 단위 테스트 파일이다. `Renderer` 디렉티브를 인스턴스 추적 `MockRenderer`로 교체하여 정렬/분배 CSS 클래스 적용과 자식 컴포넌트 렌더링 동작을 검증한다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture`, `TestBed`
- `@angular/core` — `Directive`, `Input`

### 저장소 내부 모듈
- [`./column`](./column.ts.md) — `Column` (테스트 대상)
- [`../data/processor`](../data/processor.ts.md) — `MessageProcessor`
- [`../rendering/theming`](../rendering/theming.ts.md) — `Theme`
- [`../rendering/catalog`](../rendering/catalog.ts.md) — `Catalog`
- `../types` (타입 전용) — `ColumnNode`

## 테스트 케이스

### 내부 클래스: `MockRenderer`
- `@Directive({ selector: '[a2ui-renderer]', standalone: true })`로 선언된 테스트 전용 가짜 디렉티브.
- `@Input() surfaceId: string`, `@Input() component: any`를 가진다.
- `static instances: MockRenderer[]` 배열로 생성된 인스턴스 수를 추적한다.
- 생성자에서 `MockRenderer.instances.push(this)`를 실행한다.

### 픽스처 및 모킹 설정 (`beforeEach`)
- `mockTheme`: `components.Column: {'custom-col': true}`로 설정한 실제 `Theme` 인스턴스.
- `mockNode`: `id: 'col-1'`, `type: 'Column'`, `weight: 1`, `properties.children: [{id: 'child-1', type: 'Text', properties: {}}]`를 가진 `ColumnNode` 픽스처.
- `TestBed.overrideComponent`의 `set.imports`를 `[MockRenderer]`로 지정하여 `Renderer`를 교체한다.
- `beforeEach` 내에서 `MockRenderer.instances = []`로 인스턴스 추적 배열을 초기화한다.
- 입력: `surfaceId: 'surface-1'`, `component: mockNode`, `weight: 1`.

### 테스트: `'should create'`
- 검증: `component`가 truthy임을 확인.

### 테스트: `'should apply alignment and distribution classes'`
- 검증 1 (기본값): `<section>` 요소의 `className`에 `'align-stretch'`와 `'distribute-start'`가 포함됨을 확인한다. 이는 `alignment` 기본값 `'stretch'`와 `distribution` 기본값 `'start'`에 대응한다.
- 동작: `alignment` 입력을 `'center'`로, `distribution` 입력을 `'end'`로 변경하고 `detectChanges()`.
- 검증 2 (변경 후): `className`에 `'align-center'`와 `'distribute-end'`가 포함됨을 확인한다.
- 경계 케이스: `classes()` 계산 신호가 입력 변경 시 올바르게 재계산됨을 검증.

### 테스트: `'should render child components'`
- 검증: `MockRenderer.instances.length`가 `1`임을 확인하여 `mockNode.properties.children`에 있는 단일 자식이 렌더링되었음을 검증한다.

## 동작 흐름

`beforeEach`에서 `TestBed`를 구성하고 `Renderer`를 `MockRenderer`로 교체한다. 인스턴스 추적 배열을 매 테스트 전에 초기화한다. 클래스 테스트는 `setInput`으로 `alignment`와 `distribution` 입력을 동적으로 변경하고 `detectChanges()` 후 DOM의 `className`을 확인한다. 자식 렌더링 테스트는 `MockRenderer.instances.length`로 렌더링된 자식 수를 직접 확인한다.
