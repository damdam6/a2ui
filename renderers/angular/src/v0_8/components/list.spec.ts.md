# renderers/angular/src/v0_8/components/list.spec.ts

## 개요

`List` 컴포넌트에 대한 Angular 단위 테스트 파일이다. 방향 속성 반영, 자식 컴포넌트 목록 렌더링, `.a2ui-list-item` 래퍼 생성을 검증한다. `Renderer` 디렉티브 대신 `MockRenderer`를 사용해 자식 렌더링 인스턴스 수를 추적한다.

## 의존성

### 외부 패키지
- `@angular/core/testing`: `ComponentFixture`, `TestBed`
- `@angular/core`: `Directive`, `Input`
- `@angular/platform-browser`: `By`

### 저장소 내부 모듈
- [`./list`](./list.ts.md): 테스트 대상 `List` 컴포넌트
- [`../data/processor`](../data/processor.ts.md): `MessageProcessor` 서비스
- [`../rendering/theming`](../rendering/theming.ts.md): `Theme` 서비스
- [`../rendering/catalog`](../rendering/catalog.ts.md): `Catalog` 서비스
- [`../types`](../types.ts.md): `ListNode` 타입

## Exports

없음 (테스트 파일).

## 테스트 케이스 명세

### 모킹 클래스: `MockRenderer`

`selector: '[a2ui-renderer]'`인 standalone `Directive`. `surfaceId`와 `component` `@Input`을 가지며, `static instances: MockRenderer[]` 배열에 생성된 인스턴스를 자동 추적한다. `List`의 `overrideComponent`를 통해 실제 `Renderer` 대신 주입된다.

### 픽스처 / 모킹 설정 (`beforeEach`)

- `mockTheme`: `new Theme()` 후 `components`를 `{List: 'list-class'}` 로 설정.
- `mockNode`: `ListNode`, `{id: 'list-1', type: 'List', weight: 1, properties: {children: [{id: 'child-1', type: 'Text', properties: {}}]}}`.
- `TestBed` 설정에서 `overrideComponent(List, {set: {imports: [MockRenderer]}})` 로 Renderer를 MockRenderer로 교체.
- `TestBed` 제공자: `MessageProcessor → {}`, `Theme → mockTheme`, `Catalog → {}`.
- 각 `beforeEach` 시작 시 `MockRenderer.instances = []` 로 인스턴스 추적 초기화.
- 입력값 설정: `surfaceId: 'surface-1'`, `component: mockNode`, `weight: 1`.

### 테스트 케이스

| 케이스명 | 검증 동작 | 픽스처/모킹 |
|----------|-----------|-------------|
| `should create` | 컴포넌트 인스턴스가 truthy인지 확인 | 기본 설정 |
| `should apply direction attribute` | 기본 상태에서 호스트의 `direction` 속성이 `'vertical'`, `setInput('direction', 'horizontal')` 후 `'horizontal'`인지 확인 | `setInput('direction', ...)` |
| `should render child components wrapped in list-item` | `MockRenderer.instances.length`가 1(자식 하나)이고, `.a2ui-list-item` 클래스 요소가 DOM에 존재하는지 확인 | `MockRenderer.instances` 추적 |

## 동작 흐름

각 테스트는 `overrideComponent`로 교체된 `MockRenderer`를 통해 자식 렌더링 횟수를 추적하고, `getAttribute('direction')` 및 `debugElement.query`로 DOM 상태를 검증한다.
