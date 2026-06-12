# renderers/angular/src/v0_8/v0_8_integration.spec.ts

## 개요

v0.8 Angular 렌더러의 통합 테스트 파일이다. 실제 카탈로그(`DEFAULT_CATALOG`)와 실제 Angular 컴포넌트들을 함께 사용하여, 스타일 주입, v0.8 래핑 형식 언래핑, `Text`/`Card`/`Row`/`Image` 등 복합 컴포넌트 트리 렌더링, 알 수 없는 속성 graceful 처리, JSON mock 파일을 이용한 회귀 테스트까지 검증한다.

## 의존성

### 외부 패키지

- `@angular/core/testing` — `ComponentFixture`, `TestBed`
- `@angular/core` — `Component`

### 저장소 내부 모듈

- [`./rendering/renderer`](rendering/renderer.ts.md) — `Renderer`
- [`./rendering/catalog`](rendering/catalog.ts.md) — `Catalog`
- [`./catalog`](catalog/index.ts.md) — `DEFAULT_CATALOG`
- [`./rendering/theming`](rendering/theming.ts.md) — `Theme`
- `./data/processor` — `MessageProcessor`
- `./data/markdown` — `MarkdownRenderer`
- `./test_data/mocks/restaurant-card.json` — 레스토랑 카드 회귀 mock 데이터
- `./test_data/mocks/contact-card.json` — 연락처 카드 회귀 mock 데이터

## 헬퍼 함수 및 픽스처

### `resolveComponentTree(messages: any[], rootId: string): any`

v0.8 포맷의 flat 컴포넌트 메시지 배열에서 트리 구조를 재구성하는 헬퍼 함수(테스트 전용 로컬 함수).

동작 단계:
1. `messages.find(m => m.surfaceUpdate)?.surfaceUpdate`로 서피스 업데이트 메시지를 찾는다. 없으면 `null` 반환.
2. `surfaceUpdate.components`에서 `Map<id, component>` 생성.
3. 내부 재귀 함수 `resolve(idOrNode)`를 정의:
   - 문자열이면 `componentMap.get(idOrNode)`를 조회하여 재귀 호출.
   - 객체이고 `type`과 `properties`를 모두 갖추면 그대로 반환.
   - `component` 키가 있으면 `Object.keys(idOrNode.component)[0]`으로 타입 추출, `properties` 구성:
     - `properties.child`가 있으면 `resolve(properties.child)` 재귀 처리.
     - `properties.children`이 배열이면 각 요소를 `resolve`로 처리, `explicitList`가 있으면 해당 ID 배열을 `resolve`로 처리.
   - `{ id, type, properties }` 형태로 정규화하여 반환.
4. `resolve(rootId)` 결과를 반환.

### `TestHost`

- `@Component({ template: '<ng-container a2ui-renderer [surfaceId]="surfaceId" [component]="component"></ng-container>', standalone: true, imports: [Renderer] })`
- `surfaceId = 'test-surface'`, `component: any = null`

---

## 테스트 케이스

### `describe('v0.8 Angular Renderer Integration')`

**`beforeEach` 설정**
- `processor`를 `jasmine.createSpyObj('MessageProcessor', ['getData', 'dispatch', 'resolvePath', 'version'])`으로 mock.
- `processor.getData`의 기본 mock: `path === '/name'`이면 `'The Italian Kitchen'` 반환, 그 외는 `` `resolved:${path}` ``.
- `TestBed` 구성: `TestHost` 임포트, `MessageProcessor` mock 제공, `Catalog`를 `DEFAULT_CATALOG`로 제공, `MarkdownRenderer`를 `{ render: (val) => Promise.resolve(val) }`로 mock, `Theme` 제공.
- `TestBed.inject(Theme).update(...)` 호출로 `Text`, `Card`, `Row`, `Column`, `Image`, `Divider`, `Icon`, `Button`에 CSS 클래스 매핑 설정.

**`should inject structural styles and they should be effective`**
- 검증 동작: `Row` 컴포넌트를 렌더링한 후 `a2ui-row section` 요소의 `display` computed style이 `'flex'`인지 확인 (구조적 스타일이 실제로 적용됨).
- `fixture.nativeElement.querySelector('a2ui-row section')`이 존재할 때만 검증(JSDOM과 Karma 환경 차이 수용).

**`should render a Restaurant Card using real components and catalog`**
- 검증 동작: `Card → Column → Text` 형태의 중첩 컴포넌트 트리를 실제 카탈로그로 렌더링할 때, `a2ui-text section`에 데이터 경로 `/name`이 `'The Italian Kitchen'`으로 해석된 값이 표시되고, `usageHint: 'h2'`에 따라 `a2ui-text-h2` CSS 클래스가 적용된다.
- `processor.getData` mock: `path === '/name'` → `'The Italian Kitchen'`.

**`should handle the v0.8 component wrapper format (unwrapping)`**
- 검증 동작: `{ id, component: { Text: { text: { literalString: 'Wrapped Text' } } } }` 형태의 v0.8 래핑 형식을 `Renderer`가 자동으로 언래핑하여 `a2ui-text section`에 `'Wrapped Text'`를 렌더링한다.

**`should be resilient to unknown component properties`**
- 검증 동작: `Text` 컴포넌트 노드에 선언되지 않은 속성(`unknownProp: 'should not crash'`)이 있어도 `detectChanges()`가 에러를 던지지 않고 `console.warn`이 호출된다.
- `spyOn(console, 'warn')` 모킹 후 `not.toThrow()` 검증.

---

### `describe('Regression Mocks')`

두 테스트 모두 동일한 패턴을 따른다:
1. JSON mock 파일(`restaurantCardMock` / `contactCardMock`)을 로드하여 `beginRendering` 및 `dataModelUpdate` 메시지를 파싱.
2. `resolveComponentTree`로 컴포넌트 트리를 재구성.
3. `dataModelUpdate.contents`에서 `Map<key, value>` 데이터 모델 구성, `processor.getData` mock 재설정.
4. `fixture.componentInstance.component = componentTree` 설정 후 `detectChanges` → `whenStable` → `detectChanges`.

**`should render the Restaurant Card regression mock correctly`**
- `a2ui-card` 요소 존재 확인.
- `a2ui-image img` 요소 존재 및 `src`에 `'unsplash.com'` 포함 확인.
- `a2ui-text section`에 `'The Italian Kitchen'` 포함 확인.
- `a2ui-row` 요소가 3개 이상 존재 확인.

**`should render the Contact Card regression mock correctly`**
- `a2ui-card` 요소 존재 확인.
- `a2ui-text section`에 `'David Park'` 포함 확인.
- `a2ui-image img` 요소 존재 및 `src`에 `'unsplash.com'` 포함 확인.
- `a2ui-divider` 요소 존재 확인.
