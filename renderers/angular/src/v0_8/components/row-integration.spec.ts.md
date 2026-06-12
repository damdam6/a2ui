# renderers/angular/src/v0_8/components/row-integration.spec.ts

## 개요

`Row` 컴포넌트의 통합 테스트 파일이다. 실제 `Renderer` 디렉티브와 `DEFAULT_CATALOG`를 사용하는 `TestHost` 컴포넌트를 통해 `Row`가 실제 렌더링 파이프라인에서 올바르게 동작하는지 검증한다. 단위 테스트와 달리 모의(Mock) 렌더러를 사용하지 않고 실제 컴포넌트 조합을 테스트한다.

## 의존성

### 외부 패키지
- `@angular/core/testing`: `ComponentFixture`, `TestBed`
- `@angular/core`: `Component`

### 저장소 내부 모듈
- [`../rendering/renderer`](../rendering/renderer.ts.md) — `Renderer` 디렉티브
- [`../rendering/catalog`](../rendering/catalog.ts.md) — `Catalog`
- [`../catalog`](../catalog.ts.md) — `DEFAULT_CATALOG`
- [`../rendering/theming`](../rendering/theming.ts.md) — `Theme`
- [`../data/processor`](../data/processor.ts.md) — `MessageProcessor` (모킹 대상)
- [`../data/markdown`](../data/markdown.ts.md) — `MarkdownRenderer`

## Exports

없음 (테스트 파일)

## 테스트 케이스

### `TestHost` 헬퍼 컴포넌트
`@Component` 데코레이터로 정의된 인라인 standalone 컴포넌트. `a2ui-renderer` 디렉티브를 사용하는 `<ng-container>` 템플릿을 가지며, `surfaceId`(초기값 `'test-surface'`)와 `component`(초기값 `null`) 속성을 노출한다.

### 픽스처 및 모킹 설정 (`beforeEach`)
- `processor`: `jasmine.createSpyObj`로 `getData`, `dispatch`, `resolvePath`, `version` 메서드를 스파이로 생성.
- `TestBed` 구성: `TestHost` 임포트, `MessageProcessor`(스파이), `Catalog`(`DEFAULT_CATALOG`), `MarkdownRenderer`(`render` 함수는 `Promise.resolve(val)` 반환), `Theme` 프로바이더 주입.
- `theme.update()`로 테마를 구성: `Row: {'a2ui-row': true}`, `Text: {all: {'a2ui-text': true}}`. `markdown` 섹션에 `p, h1, h2, h3, h4, h5, ul, ol, li, a, strong, em` 키를 빈 배열로 초기화.

---

### `'should render children using the real renderer'`
- **검증 동작**: `Row` 타입 컴포넌트(`id:'row-1'`)에 두 개의 `Text` 자식(`id:'child-1'`, `id:'child-2'`)을 설정하고, `fixture.whenStable()` 대기 후 실제 DOM에서 `a2ui-row` 요소가 존재하는지, `a2ui-text` 요소가 2개이고 각각 `'Child 1'`, `'Child 2'` 텍스트를 포함하는지 확인.
- **픽스처**: `fixture.componentInstance.component`에 component tree 직접 할당. `detectChanges()`, `whenStable()`, `detectChanges()` 순서로 호출하여 비동기 처리 완료를 보장.

---

### `'should pass alignment and distribution to the real Row component'`
- **검증 동작**: `Row` 컴포넌트에 `alignment: 'center'`, `distribution: 'space-between'`을 설정했을 때, 렌더링된 `a2ui-row section` 요소의 `className`에 `'align-center'`와 `'distribute-space-between'`이 포함되는지 확인.
- **픽스처**: `fixture.nativeElement.querySelector('a2ui-row section')`으로 DOM 쿼리.

## 동작 흐름

`TestHost`를 통해 실제 렌더링 파이프라인을 거치기 때문에 `DEFAULT_CATALOG`에 등록된 컴포넌트들이 실제로 인스턴스화된다. `whenStable()` 호출로 비동기 초기화(마크다운 렌더링 등)가 완료될 때까지 대기한 뒤 DOM 검증을 수행한다.
