# renderers/angular/src/v0_8/components/modal.spec.ts

## 개요

`Modal` 컴포넌트에 대한 Angular 단위 테스트 파일이다. 모달의 열림/닫힘 상태 전환, 엔트리 포인트 렌더링, 오버레이·백드롭·콘텐츠 영역의 조건부 표시, 백드롭 클릭으로 닫기, 모달 내부 클릭 시 닫히지 않는 동작을 검증한다. `Renderer` 디렉티브를 `MockRenderer`로 교체해 자식 렌더링 인스턴스를 추적한다.

## 의존성

### 외부 패키지
- `@angular/core/testing`: `ComponentFixture`, `TestBed`
- `@angular/core`: `Directive`, `Input`
- `@angular/platform-browser`: `By`

### 저장소 내부 모듈
- [`./modal`](./modal.ts.md): 테스트 대상 `Modal` 컴포넌트
- [`../data/processor`](../data/processor.ts.md): `MessageProcessor` 서비스
- [`../rendering/theming`](../rendering/theming.ts.md): `Theme` 서비스
- [`../rendering/catalog`](../rendering/catalog.ts.md): `Catalog` 서비스
- [`../types`](../types.ts.md): `AnyComponentNode` 타입

## Exports

없음 (테스트 파일).

## 테스트 케이스 명세

### 모킹 클래스: `MockRenderer`

`selector: '[a2ui-renderer]'`인 standalone `Directive`. `surfaceId`와 `component` `@Input`을 보유하며, `static instances: MockRenderer[]`로 생성된 인스턴스를 자동 추적한다. `overrideComponent`로 실제 `Renderer`를 대체한다.

### 픽스처 / 모킹 설정 (`beforeEach`)

- `mockTheme`: `new Theme()` 후 `components.Modal`을 `{backdrop: 'backdrop-class', element: 'modal-element-class'}`로 설정.
- `mockEntryPoint`: `AnyComponentNode`, `{id: 'btn-1', type: 'Button', properties: {text: 'Open'}}`.
- `mockContent`: `AnyComponentNode`, `{id: 'text-1', type: 'Text', properties: {text: 'Hello'}}`.
- `TestBed` 설정에서 `overrideComponent(Modal, {set: {imports: [MockRenderer]}})` 로 Renderer를 MockRenderer로 교체.
- `TestBed` 제공자: `MessageProcessor → {}`, `Theme → mockTheme`, `Catalog → {}`.
- 각 `beforeEach` 시작 시 `MockRenderer.instances = []` 초기화.
- 입력값 설정: `surfaceId: 'surface-1'`, `entryPointChild: mockEntryPoint`, `contentChild: mockContent`, `component: {id: 'modal-1', type: 'Modal', weight: 1}`, `weight: 1`.

### 테스트 케이스

| 케이스명 | 검증 동작 | 픽스처/모킹 |
|----------|-----------|-------------|
| `should create` | 컴포넌트 인스턴스가 truthy인지 확인 | 기본 설정 |
| `should render entry point child initially` | 초기 상태에서 `MockRenderer.instances.length`가 1 (엔트리 포인트 하나만 렌더링)인지 확인 | `MockRenderer.instances` |
| `should not render the overlay when closed` | 초기 닫힌 상태에서 `.a2ui-modal-overlay` 요소가 DOM에 없는지 확인 | 기본 설정 |
| `should not render the backdrop when closed` | 초기 닫힌 상태에서 `.backdrop-class` 요소가 DOM에 없는지 확인 | 기본 설정 |
| `should open modal on entry point click` | `.a2ui-modal-entry-point` 클릭 후 `(component as any).isOpen()`이 true, `.backdrop-class`가 존재, `MockRenderer.instances.length`가 2인지 확인 | `.nativeElement.click()` |
| `should close modal on backdrop click` | `component.openModal()` 후 `.backdrop-class` 클릭 시 `isOpen()`이 false인지 확인 | `backdropEl.nativeElement.click()` |
| `should NOT close modal on element click` | `component.openModal()` 후 `.modal-element-class` 클릭 시 `isOpen()`이 여전히 true인지 확인 | `elementEl.nativeElement.click()` |
| `should use position fixed for overlay` | `component.openModal()` 후 `.a2ui-modal-overlay`의 computed style `position`이 `'fixed'`인지 확인 | `window.getComputedStyle` |

## 동작 흐름

각 테스트는 `overrideComponent`로 실제 렌더러를 `MockRenderer`로 대체한 상태에서 모달을 마운트하고, 클릭 이벤트 디스패치 또는 `openModal()` 공개 메서드를 통해 상태를 조작한 뒤, DOM 요소 존재 여부·CSS 속성·인스턴스 수로 기대 동작을 검증한다. `isOpen` 신호는 컴포넌트의 private 필드에 직접 접근해 검증한다.
