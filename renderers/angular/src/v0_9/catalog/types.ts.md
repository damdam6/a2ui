# renderers/angular/src/v0_9/catalog/types.ts

## 개요

Angular 렌더러에서 사용하는 카탈로그 관련 타입을 정의하는 모듈이다. A2UI 프로토콜의 `ComponentApi` 및 `Catalog`를 Angular 전용으로 확장하여, 각 컴포넌트 구현에 Angular `Type` 참조를 연결하는 인터페이스와 클래스를 제공한다. 또한 스키마 정렬 과정 중 임시로 타입 검사를 우회하기 위한 `AnyDuringSchemaAlignment` 타입 별칭도 포함한다.

## 의존성

### 외부 패키지
- `@angular/core`: `Type`
- `@a2ui/web_core/v0_9`: `Catalog`, `ComponentApi`

### 저장소 내부 모듈
- [`../core/catalog_component_instance`](../core/catalog_component_instance.ts.md)

## Exports

| 이름 | 종류 | 설명 |
|------|------|------|
| `AnyDuringSchemaAlignment` | 타입 별칭 | `any`의 별칭. 스키마 정렬 완료 전까지 임시로 사용. |
| `AngularComponentImplementation` | 인터페이스 | `ComponentApi`를 확장하며 Angular 컴포넌트 클래스 참조를 추가. |
| `AngularCatalog` | 클래스 | `Catalog<AngularComponentImplementation>`을 상속하는 Angular 전용 카탈로그. |

## 상세 명세

### `AnyDuringSchemaAlignment`

`any` 타입의 별칭이다. 기본 카탈로그 스키마와 Angular 구현 간의 타입 정렬이 완료될 때까지 엄격한 타입 검사를 우회하기 위한 임시 타입 별칭이다. GitHub issue #1303 해결 후 제거 예정.

---

### `AngularComponentImplementation`

`ComponentApi` 인터페이스를 확장하는 인터페이스.

**필드**:

| 이름 | 타입 | 설명 |
|------|------|------|
| `component` (readonly) | `Type<CatalogComponentInstance>` | 이 컴포넌트를 렌더링하는 Angular 컴포넌트 클래스. `props`, `surfaceId`, `dataContextPath`를 입력으로 받는 standalone 컴포넌트여야 한다. |

---

### `AngularCatalog`

**상속**: `Catalog<AngularComponentImplementation>`

`Catalog` 제네릭 클래스에 `AngularComponentImplementation` 타입 파라미터를 적용한 구체 클래스. 별도의 추가 로직 없이 타입 특화만 수행한다. `MessageProcessor`가 컴포넌트 정의를 조회하고 `ComponentHostComponent`가 올바른 Angular 컴포넌트를 인스턴스화할 때 이 카탈로그를 사용한다.

## 동작 흐름

이 파일은 런타임 로직이 없는 타입 정의 모듈이다. `AngularCatalog` 인스턴스는 컴포넌트 타입 문자열 키를 `AngularComponentImplementation` 객체(Angular `Type` 참조 포함)에 매핑하는 레지스트리로 동작한다. 렌더러 서비스 초기화 시 `A2uiRendererService`에 주입되고, `ComponentHostComponent`는 이 카탈로그로부터 렌더링할 컴포넌트 클래스를 조회한다.
