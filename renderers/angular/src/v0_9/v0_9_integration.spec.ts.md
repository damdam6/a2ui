# renderers/angular/src/v0_9/v0_9_integration.spec.ts

## 개요

A2UI Angular 렌더러 v0.9의 엔드투엔드 통합 테스트 파일이다. `A2uiRendererService.processMessages()`를 통해 실제 A2UI 프로토콜 메시지를 처리하고, Angular DOM에 올바른 컴포넌트 트리가 렌더링되는지 검증한다. 데이터 바인딩의 리액티브 업데이트, 액션 핸들러 호출, 그리고 레스토랑 카드/연락처 카드 회귀 목업 렌더링을 포함한 5개 테스트 케이스로 구성된다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture`, `TestBed`
- `@angular/core` — `Component`, `ChangeDetectionStrategy`
- `@a2ui/web_core/v0_9` — `A2uiMessage`

### 저장소 내부 모듈
- [`./core/a2ui-renderer.service`](./core/a2ui-renderer.service.ts.md) — `A2uiRendererService`, `A2UI_RENDERER_CONFIG`
- [`./core/surface.component`](./core/surface.component.ts.md) — `SurfaceComponent`
- [`./catalog/basic/basic-catalog`](./catalog/basic/basic-catalog.ts.md) — `BasicCatalog`
- [`./core/markdown`](./core/markdown.ts.md) — `MarkdownRenderer`
- `./test_data/mocks/restaurant-card.json` — 레스토랑 카드 목업 메시지 데이터
- `./test_data/mocks/contact-card.json` — 연락처 카드 목업 메시지 데이터

## Exports

없음 (테스트 전용 파일).

## 테스트 케이스 명세

### `@Component TestHost` (테스트용 인라인 컴포넌트)

- 템플릿: `<div id="test-host">` 내부에 `<a2ui-v09-surface [surfaceId]="surfaceId">` 렌더링
- `standalone: true`, `ChangeDetectionStrategy.OnPush`
- `surfaceId` 필드 초기값: `'test-surface'`

---

### `describe('v0.9 Angular Renderer Integration')`

#### `beforeEach`

- `actionSpy`: 액션 핸들러 스파이 생성.
- `TestBed.configureTestingModule` 설정:
  - `A2uiRendererService`, `BasicCatalog` 직접 제공.
  - `A2UI_RENDERER_CONFIG`: `BasicCatalog`를 의존성으로 주입받아 `{ catalogs: [basicCatalog], actionHandler: actionSpy }` 반환.
  - `MarkdownRenderer`: `{ render: (val: string) => Promise.resolve(val) }` (동기식 패스스루 목).
- `TestHost` 픽스처 및 `A2uiRendererService` 인스턴스 획득.

---

#### `it('should render a basic component tree from protocol messages')`

**검증 동작**: `createSurface` + `updateComponents` 메시지를 처리하면 `Column → Text + Button(Text)` 트리가 DOM에 올바르게 렌더링된다.

**픽스처**: `version: 'v0.9'`의 `createSurface` 메시지(catalogId: `'https://a2ui.org/specification/v0_9/catalogs/basic/catalog.json'`) 후 `updateComponents` 메시지로 `root(Column)`, `text-id(Text: 'Hello v0.9')`, `button-id(Button)`, `button-text-id(Text: 'Click Me')` 컴포넌트 등록.

**검증 단계**:
1. `a2ui-v09-column` 요소가 존재해야 한다.
2. `a2ui-v09-text`가 존재하고 텍스트에 `'Hello v0.9'`를 포함해야 한다.
3. `a2ui-v09-button button`이 존재하고 텍스트에 `'Click Me'`를 포함해야 한다.

---

#### `it('should handle data model updates and reactive data binding')`

**검증 동작**: `updateDataModel` 메시지로 데이터 모델을 업데이트하면 데이터 바인딩된 텍스트 컴포넌트가 새 값을 렌더링한다.

**픽스처**: `root(Text)` 컴포넌트의 `text` 속성이 `{ path: '/user/name' }`으로 바인딩됨.

**검증 단계**:
1. 초기 상태: `a2ui-v09-text`의 텍스트가 빈 문자열이어야 한다.
2. `updateDataModel`로 `/user/name`을 `'Alice'`로 설정.
3. 변경 감지 후 `a2ui-v09-text`의 텍스트가 `'Alice'`를 포함해야 한다.

---

#### `it('should dispatch actions to the action handler')`

**검증 동작**: 버튼 클릭 시 `actionHandler`가 올바른 인자로 호출된다.

**픽스처**: `root(Button)`에 `action: { event: { name: 'navigate', context: { url: 'https://example.com' } } }` 설정.

**검증 단계**:
1. `button` 요소를 찾아 `.click()` 호출.
2. `actionSpy`가 호출되어야 한다.
3. 호출 인자(`actionArg`)가 다음을 만족해야 한다:
   - `actionArg.surfaceId === 'test-surface'`
   - `actionArg.name === 'navigate'`
   - `actionArg.context` deep equal `{ url: 'https://example.com' }`
   - `actionArg.sourceComponentId === 'root'`
   - `actionArg.timestamp`가 정의되어야 한다.

---

### `describe('Regression Mocks')`

#### `it('should render the Restaurant Card regression mock correctly')`

**검증 동작**: `restaurant-card.json` 목업 메시지로 서피스를 렌더링하면 카드, 이미지, 텍스트, 행 요소들이 올바르게 표시된다.

**픽스처**: `restaurantCardMock` JSON 파일 메시지 처리. `surfaceId = 'gallery-restaurant-card'`.

**검증 단계**:
1. `a2ui-v09-card` 존재 확인.
2. `a2ui-v09-image img` 존재 + `src`에 `'unsplash.com'` 포함.
3. 첫 번째 `a2ui-v09-text`의 텍스트에 `'The Italian Kitchen'` 포함.
4. `a2ui-v09-row` 요소가 3개 이상 존재.

---

#### `it('should render the Contact Card regression mock correctly')`

**검증 동작**: `contact-card.json` 목업 메시지로 서피스를 렌더링하면 카드, 이름 텍스트, 아바타 이미지, 구분선이 올바르게 표시된다.

**픽스처**: `contactCardMock` JSON 파일 메시지 처리. `surfaceId = 'gallery-contact-card'`.

**검증 단계**:
1. `a2ui-v09-card` 존재 확인.
2. 첫 번째 `a2ui-v09-text`에 `'David Park'` 포함.
3. `a2ui-v09-image img` 존재 + `src`에 `'unsplash.com'` 포함.
4. `a2ui-v09-divider` 존재 확인.

## 동작 흐름

모든 테스트는 `TestHost` 컴포넌트를 통해 실제 Angular 렌더링 파이프라인을 거친다. `rendererService.processMessages()`로 A2UI 프로토콜 메시지를 주입하고, `fixture.detectChanges()` + `fixture.whenStable()` 조합으로 비동기 초기화를 완료한 뒤, `nativeElement.querySelector`로 DOM을 직접 검사한다. 회귀 목업 테스트는 JSON 파일에서 실제 서비스가 전송할 메시지 형식을 그대로 불러와 사용한다.
