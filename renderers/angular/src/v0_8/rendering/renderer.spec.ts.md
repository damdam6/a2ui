# renderers/angular/src/v0_8/rendering/renderer.spec.ts

## 개요

`Renderer` 지시자의 단위 테스트 및 회귀 테스트 파일이다. 두 개의 `describe` 블록으로 구성되어 있으며, 카탈로그 기반 컴포넌트 렌더링, 비동기 컴포넌트 해석, 알 수 없는 타입 에러 처리, 스타일 주입, 누락된 입력에 대한 graceful 처리를 검증한다.

## 의존성

### 외부 패키지

- `@angular/core/testing` — `ComponentFixture`, `TestBed`
- `@angular/core` — `Component`, `Input`

### 저장소 내부 모듈

- [`./renderer`](renderer.ts.md) — `Renderer`
- [`./catalog`](catalog.ts.md) — `Catalog`

## 픽스처 및 헬퍼 컴포넌트

### `TestComp`

- `@Component({ selector: 'test-comp', template: '<div>{{ text }}</div>', standalone: true })`
- `@Input() surfaceId?: string`, `@Input() component?: any`, `@Input() weight?: number`, `@Input() text?: string`
- `Renderer`가 카탈로그에서 조회해 동적으로 렌더링하는 테스트용 컴포넌트.

### `TestHost`

- `@Component({ template: '<ng-container a2ui-renderer [surfaceId]="surfaceId" [component]="component"></ng-container>', standalone: true, imports: [Renderer] })`
- `surfaceId = ''`, `component: any = null`
- 두 `describe` 블록 모두에서 사용되는 호스트 컴포넌트. `Renderer` 지시자를 `ng-container`에 적용한다.

### `CompWithInputs`

- `@Component({ selector: 'comp-with-inputs', template: '', standalone: true })`
- `@Input() surfaceId`, `@Input() component`, `@Input() weight`, `@Input() text` 선언. `children` 및 `child` 입력은 없다.
- 회귀 테스트에서 누락 입력 경고를 검증하는 데 사용.

---

## 테스트 케이스

### `describe('v0.8 Renderer')`

**`beforeEach` 픽스처 설정**
- `mockCatalog = { TestComp: { type: () => TestComp } }` 형태로 mock 카탈로그 구성.
- `TestBed.configureTestingModule`에 `TestHost`를 임포트하고 `{ provide: Catalog, useValue: mockCatalog }` 제공자 등록.
- `fixture = TestBed.createComponent(TestHost)` 생성.

**`should render component from catalog`**
- 검증 동작: 카탈로그에 등록된 컴포넌트 타입을 `type: 'TestComp'`으로 지정하면 해당 컴포넌트가 렌더링되어 `properties.text` 값(`'Hello v0.8'`)이 DOM에 나타난다.
- `surfaceId = 'surf-1'`, `component = { type: 'TestComp', properties: { text: 'Hello v0.8' }, weight: 10 }` 설정 후 `detectChanges` → `whenStable` → `detectChanges` 순서.
- `expect(compiled.textContent).toContain('Hello v0.8')` 검증.

**`should handle async component resolution`**
- 검증 동작: 카탈로그 항목을 `() => Promise.resolve(TestComp)` 형태(비동기 로더)로 교체해도 최종적으로 컴포넌트가 렌더링된다.
- `mockCatalog['TestComp'] = () => Promise.resolve(TestComp)` 교체, `text: 'Async Hello'`로 설정.
- `whenStable()` 완료 후 DOM에 `'Async Hello'` 포함 여부 검증.

**`should error if component type not found`**
- 검증 동작: 카탈로그에 없는 타입(`'UnknownComp'`)이 주어지면 `console.error`로 `'Unknown component type: UnknownComp'`가 출력된다.
- `spyOn(console, 'error')` 모킹 후 `detectChanges` 실행.

**`should handle direct function config in catalog`**
- 검증 동작: 카탈로그 항목이 `() => TestComp`(Promise 없이 동기 반환하는 함수)일 때도 렌더링이 정상 작동한다.
- `text: 'Function Hello'` 설정, `whenStable` 후 DOM 검증.

**`should inject structural styles into the document head`**
- 검증 동작: `Renderer`가 초기화될 때 `structuralStyles`를 포함하는 `<style>` 태그가 `document.head`에 주입된다.
- `document.getElementsByTagName('style')`로 모든 스타일 태그를 순회하여 `.layout-p-` 클래스가 포함된 태그가 하나 이상 존재하는지 확인.

---

### `describe('v0.8 Renderer Regression Tests')`

**`beforeEach` 픽스처 설정**
- `mockCatalog = { CompWithInputs: { type: () => CompWithInputs } }` 구성.
- 동일한 `TestHost` 기반으로 `TestBed` 구성.

**`should gracefully handle missing inputs and log a warning`**
- 검증 동작: 컴포넌트 노드의 `properties`에 존재하지만 컴포넌트 클래스에 `@Input()`으로 선언되지 않은 속성(`nonExistentInput`)이 있을 때 `NG0303` 에러를 던지지 않고 `console.warn`으로 경고 메시지를 출력한다.
- `spyOn(console, 'warn')` 모킹. 경고 메시지 패턴: `/Property "nonExistentInput" could not be set on component CompWithInputs/`.

**`should pass children and child properties as inputs if supported`**
- 검증 동작: 컴포넌트가 `children` 및 `child` setter 입력을 선언하면 노드의 `properties.children`과 `properties.child`가 각각 해당 setter로 전달된다.
- 인라인으로 `CompWithChildren` 컴포넌트를 정의(set 메서드로 `setCapture`에 캡처). `children: [{id: 'child-1'}]`, `child: {id: 'child-2'}` 설정.
- `expect(setCapture.children).toEqual([{id: 'child-1'}])` 및 `expect(setCapture.child).toEqual({id: 'child-2'})` 검증.

**`should gracefully handle components with missing properties`**
- 검증 동작: 노드에 `properties` 키가 없어도 `detectChanges()` 호출 시 예외가 발생하지 않는다.
- `component = { type: 'CompWithInputs' }` (properties 없음) 설정 후 `expect(() => { fixture.detectChanges(); }).not.toThrow()` 검증.
