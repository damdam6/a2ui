# renderers/angular/src/v0_8/components/row.spec.ts

## 개요

`Row` 컴포넌트의 단위 테스트 파일이다. `Renderer` 디렉티브를 `MockRenderer`로 교체하여 자식 렌더링 호출 횟수를 추적하고, 정렬(alignment) 및 배포(distribution) CSS 클래스 적용 동작을 검증한다.

## 의존성

### 외부 패키지
- `@angular/core/testing`: `ComponentFixture`, `TestBed`
- `@angular/core`: `Directive`, `Input`

### 저장소 내부 모듈
- [`./row`](./row.ts.md) — 테스트 대상 `Row` 컴포넌트
- [`../data/processor`](../data/processor.ts.md) — `MessageProcessor` (빈 객체로 대체)
- [`../rendering/theming`](../rendering/theming.ts.md) — `Theme`
- [`../rendering/catalog`](../rendering/catalog.ts.md) — `Catalog`
- [`../types`](../types.ts.md) — `RowNode` 타입

## Exports

없음 (테스트 파일)

## 테스트 케이스

### `MockRenderer` 디렉티브
`selector: '[a2ui-renderer]'`인 standalone 디렉티브. `surfaceId: string`과 `component: any` `@Input`을 가진다. 정적 배열 `MockRenderer.instances`에 생성된 인스턴스를 모두 추적한다.

### 픽스처 및 모킹 설정 (`beforeEach`)
- `mockTheme`: `new Theme()`. `components.Row`를 `{'custom-row': true}`로 설정.
- `mockNode`: `RowNode` 타입. `id:'row-1'`, `type:'Row'`, `weight:1`, `properties.children`에 `[{id:'child-1', type:'Text', properties:{}}]` 1개.
- `TestBed` 구성: `Row` 임포트, `MessageProcessor`(빈 객체), `Theme`(mockTheme), `Catalog`(빈 객체) 프로바이더.
- `overrideComponent(Row, {set: {imports: [MockRenderer]}})`: `Row` 내부의 `Renderer`를 `MockRenderer`로 교체.
- `MockRenderer.instances = []` 초기화 후 `componentRef.setInput`으로 `surfaceId='surface-1'`, `component=mockNode`, `weight=1` 설정.

---

### `'should create'`
- **검증**: 컴포넌트 인스턴스가 truthy인지 확인.

---

### `'should apply alignment and distribution classes'`
- **검증 동작**:
  1. 기본값 검증: `section` 요소의 `className`에 `'align-stretch'`와 `'distribute-start'`가 포함되는지 확인.
  2. 값 변경 후 검증: `alignment='center'`, `distribution='end'`로 입력을 변경하고 `detectChanges()` 후 `'align-center'`, `'distribute-end'`가 포함되는지 확인.
- **픽스처**: `fixture.nativeElement.querySelector('section')`으로 DOM 쿼리.

---

### `'should render child components'`
- **검증 동작**: `MockRenderer.instances.length`가 `1`인지 확인하여 1개의 자식 노드에 대해 렌더러가 1번 인스턴스화되었음을 검증.

## 동작 흐름

`MockRenderer`를 통해 자식 렌더링 호출을 인터셉트하고 CSS 클래스를 직접 DOM에서 검증한다. `beforeEach`마다 `MockRenderer.instances`를 초기화하여 테스트 간 격리를 보장한다.
