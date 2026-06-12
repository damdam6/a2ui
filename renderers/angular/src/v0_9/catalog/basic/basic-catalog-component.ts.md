# renderers/angular/src/v0_9/catalog/basic/basic-catalog-component.ts

## 개요

Angular v0.9 basic catalog 컴포넌트 전용 추상 기반 클래스를 정의한다. 인스턴스화 시 `injectBasicCatalogStyles()`를 자동 호출하여 공통 CSS를 주입하고, 호스트 엘리먼트에 `flex` 스타일과 `--a2ui-color-primary` CSS 변수를 바인딩하여 테마 색상과 레이아웃 가중치를 반영한다. 모든 basic catalog 컴포넌트(ButtonComponent, CardComponent 등)는 이 클래스를 상속한다.

## 의존성

### 외부 패키지
- `@angular/core` — `Directive`, `computed`, `HostBinding`, `inject`
- `@a2ui/web_core/v0_9/basic_catalog` — `injectBasicCatalogStyles`
- `@a2ui/web_core/v0_9` — `ComponentApi`

### 저장소 내부 모듈
- [`../../core/a2ui-renderer.service`](../../core/a2ui-renderer.service.ts.md)
- [`../../core/catalog_component`](../../core/catalog_component.ts.md)
- [`../../core/types`](../../core/types.ts.md)

## Exports

- `BasicCatalogComponent<Api>` — 추상 클래스 (제네릭, `CatalogComponent<Api>` 상속)

## 상세 명세

### `BasicCatalogComponent<Api extends ComponentApi>`

`@Directive()` 데코레이터가 붙은 추상 클래스. 타입 파라미터 `Api`는 `ComponentApi`를 상한으로 한다.

#### 필드 및 computed 시그널

| 이름 | 타입 | 설명 |
|---|---|---|
| `rendererService` | `A2uiRendererService` | `inject(A2uiRendererService)`로 주입된 렌더러 서비스 |
| `surface` | `Signal<Surface \| undefined>` | `rendererService.surfaceGroup.getSurface(this.surfaceId())`를 호출해 현재 서피스를 반환하는 computed |
| `theme` | `Signal<Theme \| undefined>` | `surface()?.theme`를 반환하는 computed |
| `primaryColor` | `Signal<string \| undefined>` | `theme()?.primaryColor`를 반환하는 computed |
| `weight` (protected) | `Signal<number \| null>` | `props()`를 `{weight?: BoundProperty<number \| undefined>}`로 캐스팅해 `props()['weight']?.value() ?? null`을 반환하는 computed |

#### 생성자

`super()`를 호출한 뒤 `injectBasicCatalogStyles()`를 실행하여 basic catalog 전용 CSS를 전역 주입한다.

#### `get flexStyle(): string | null`

`@HostBinding('style.flex')`가 적용된 getter. `weight()`가 `null`이 아니면 `"${weight()}"` 문자열을 반환하고, `null`이면 `null`을 반환하여 바인딩을 제거한다.

#### `get primaryColorStyle(): string | null`

`@HostBinding('style.--a2ui-color-primary')`가 적용된 getter. `primaryColor()`가 truthy이면 해당 값을 반환하고 falsy이면 `null`을 반환한다. 이 바인딩을 통해 호스트 엘리먼트에 CSS 변수가 인라인으로 설정되어 자식 컴포넌트의 primary 색상이 테마에 따라 재정의된다.

## 동작 흐름

컴포넌트가 생성될 때 생성자가 `injectBasicCatalogStyles()`를 호출해 스타일을 한 번만 주입한다. 이후 Angular의 변경 감지 사이클마다 `surface → theme → primaryColor` 체인의 computed 시그널이 연쇄 평가된다. `flexStyle`과 `primaryColorStyle` getter는 Angular의 Host Binding 메커니즘을 통해 호스트 DOM 엘리먼트의 인라인 스타일에 직접 반영된다.
