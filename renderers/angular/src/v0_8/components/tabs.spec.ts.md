# renderers/angular/src/v0_8/components/tabs.spec.ts

## 개요

`Tabs` 컴포넌트의 단위 테스트 파일이다. `Renderer` 디렉티브를 `MockRenderer`로 교체하여 탭 버튼 렌더링, 초기 선택 상태, 탭 클릭에 의한 선택 변경 및 컨텐츠 전환 동작을 검증한다.

## 의존성

### 외부 패키지
- `@angular/core/testing`: `ComponentFixture`, `TestBed`
- `@angular/core`: `Directive`, `Input`
- `@angular/platform-browser`: `By`

### 저장소 내부 모듈
- [`./tabs`](./tabs.ts.md) — 테스트 대상 `Tabs` 컴포넌트
- [`../data/processor`](../data/processor.ts.md) — `MessageProcessor` (부분 모킹)
- [`../rendering/theming`](../rendering/theming.ts.md) — `Theme`
- [`../rendering/catalog`](../rendering/catalog.ts.md) — `Catalog`

## Exports

없음 (테스트 파일)

## 테스트 케이스

### `MockRenderer` 디렉티브
`selector: '[a2ui-renderer]'`인 standalone 디렉티브. `surfaceId: string`와 `component: any` `@Input`을 가진다. 정적 배열 `MockRenderer.instances`에 인스턴스를 추적한다.

### 픽스처 및 모킹 설정 (`beforeEach`)
- `mockTheme`: `new Theme()`. `components.Tabs`를 `{ container: 'tabs-container', controls: { all: 'tabs-controls-all', selected: {'tabs-controls-selected': true} } }`로 설정.
- `mockTabItems`: 2개의 탭 항목. 첫 번째 `{title: {literalString:'Tab 1'}, child: {id:'child-1', type:'Text', properties:{text:'Content 1'}}}`, 두 번째 `{title: {literalString:'Tab 2'}, child: {id:'child-2', ...}}`.
- `MessageProcessor` 프로바이더: `resolvePrimitive: (p) => p?.literalString || p` 함수만 제공.
- `overrideComponent(Tabs, {set: {imports: [MockRenderer]}})`: 내부 `Renderer`를 `MockRenderer`로 교체.
- 입력값: `surfaceId='surface-1'`, `component={id:'tabs-1', type:'Tabs', weight:1}`, `weight=1`, `tabItems=mockTabItems`.

---

### `'should create'`
- **검증**: 컴포넌트 인스턴스가 truthy인지 확인.

---

### `'should render tab buttons with titles'`
- **검증 동작**: `<button>` 요소가 2개이고 각각 텍스트가 `'Tab 1'`, `'Tab 2'`인지 확인.
- **픽스처**: `By.css('button')` 쿼리.

---

### `'should initially select first tab and render its child'`
- **검증 동작**: `(component as any).selectedIndex() === 0`. `MockRenderer.instances.length === 1`이고 `instances[0].component.id === 'child-1'`. 첫 번째 버튼 className에 `'tabs-controls-selected'` 포함, 두 번째 버튼에는 미포함.

---

### `'should update selected tab on click and render new child'`
- **검증 동작**: 두 번째 버튼 클릭 후 `detectChanges()`. `selectedIndex() === 1`. `MockRenderer.instances`는 여전히 길이 1이지만(동시에 1개만 활성화) `instances[0].component.id === 'child-2'`. 첫 번째 버튼에 선택 클래스 미포함, 두 번째 버튼에 포함.
- **주의**: `MockRenderer.instances`는 클릭 이후 재렌더링 시 새 인스턴스로 교체된다. `@if` 블록이 현재 탭의 자식만 렌더링하기 때문에 항상 1개.

## 동작 흐름

`beforeEach`에서 `detectChanges()`로 초기 렌더링. 각 테스트에서 DOM 상태, MockRenderer 인스턴스 배열, 컴포넌트 내부 시그널 상태를 검증한다. 선택 클래스는 `theme.components.Tabs.controls.selected` 객체 형태(`{'tabs-controls-selected': true}`)로 Angular `[class]` 바인딩을 통해 적용된다.
