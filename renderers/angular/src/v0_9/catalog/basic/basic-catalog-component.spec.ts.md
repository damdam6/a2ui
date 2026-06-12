# renderers/angular/src/v0_9/catalog/basic/basic-catalog-component.spec.ts

## 개요

v0.9 `BasicCatalogComponent` 기반 클래스의 동작을 검증하는 단위 테스트 파일이다. 테스트 전용 `TestComponentApi`와 `TestBasicComp`를 인라인으로 정의하여, 테마 `primaryColor`가 호스트 요소의 `--a2ui-color-primary` CSS 변수에 올바르게 바인딩되는지, 그리고 테마가 없거나 `primaryColor`가 없을 때 graceful하게 처리되는지를 검증한다.

## 의존성

### 외부 패키지

- `@angular/core` — `Component`
- `@angular/core/testing` — `ComponentFixture`, `TestBed`
- `@a2ui/web_core/v0_9` — `ComponentApi`
- `zod` — `z`

### 저장소 내부 모듈

- [`./basic-catalog-component`](basic-catalog-component.ts.md) — `BasicCatalogComponent`
- `../../core/a2ui-renderer.service` — `A2uiRendererService`, `A2UI_RENDERER_CONFIG`
- `./basic-catalog` — `BasicCatalog`

## 인라인 선언

### `TestComponentApi`

```
export const TestComponentApi = {
  name: 'TestComponent',
  schema: z.object({ foo: z.string() }).strict(),
} satisfies ComponentApi;
```

테스트 전용 `ComponentApi` 구현체. `name`은 `'TestComponent'`, `schema`는 `foo` 문자열 필드 하나를 가진 strict zod 스키마.

### `TestBasicComp`

```
@Component({ selector: 'test-basic-comp', template: '<div>Test</div>', standalone: true })
class TestBasicComp extends BasicCatalogComponent<typeof TestComponentApi> {}
```

`BasicCatalogComponent`를 상속하는 최소 테스트 컴포넌트. 별도 로직 없이 부모 클래스 동작만 검증.

---

## 테스트 케이스

### `describe('BasicCatalogComponent')`

**`beforeEach` 설정**

`TestBed.configureTestingModule` 구성:
- `imports: [TestBasicComp]`
- `providers`:
  - `A2uiRendererService` — 직접 제공 (실제 서비스 사용).
  - `BasicCatalog` — 직접 제공.
  - `A2UI_RENDERER_CONFIG` — `useFactory`로 생성. `BasicCatalog`를 주입받아 `{ catalogs: [basicCatalog] }` 설정 객체 반환. `deps: [BasicCatalog]` 선언.

`rendererService.processMessages(...)` 호출로 서피스 생성:
```js
[{
  version: 'v0.9',
  createSurface: {
    surfaceId: 'test-surface',
    catalogId: 'https://a2ui.org/specification/v0_9/catalogs/basic/catalog.json',
  },
}]
```

**`should set --a2ui-color-primary style on host`**
- 검증 동작: `surface.theme.primaryColor = '#00FF00'`으로 테마 색상을 설정한 후 `surfaceId`를 `'test-surface'`로 설정하면, 호스트 요소의 `--a2ui-color-primary` CSS 커스텀 속성 값이 `'#00FF00'`이 된다.
- `rendererService.surfaceGroup.getSurface('test-surface')` → `surface!.theme.primaryColor = '#00FF00'`
- `fixture.componentRef.setInput('surfaceId', 'test-surface')` 후 `detectChanges`.
- `element.style.getPropertyValue('--a2ui-color-primary') === '#00FF00'` 검증.

**`should handle missing theme or primaryColor`**
- 검증 동작: `primaryColor`를 설정하지 않은 상태에서 `surfaceId`를 설정하면, `--a2ui-color-primary` 속성 값이 빈 문자열(`''`)이다.
- `fixture.componentRef.setInput('surfaceId', 'test-surface')` 후 `detectChanges`.
- `element.style.getPropertyValue('--a2ui-color-primary') === ''` 검증.
