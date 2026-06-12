# renderers/angular/src/v0_8/rendering/catalog.ts

## 개요

v0.8 렌더러에서 컴포넌트 카탈로그의 타입과 Angular DI 토큰을 정의한다. 카탈로그는 컴포넌트 타입 이름(문자열 키)을 Angular 컴포넌트 클래스 또는 그 로더 함수에 매핑하는 레지스트리 역할을 하며, `Renderer`가 동적 컴포넌트를 해석할 때 이 카탈로그를 주입받아 사용한다.

## 의존성

### 외부 패키지

- `@angular/core` — `Binding`, `InjectionToken`, `Type`

### 저장소 내부 모듈

- [`./dynamic-component`](dynamic-component.ts.md) — `DynamicComponent` (제네릭 추상 지시자)
- [`../types`](../types.ts.md) — `AnyComponentNode` (타입 전용 임포트)

## Exports

| 이름 | 종류 |
|---|---|
| `CatalogLoader` | 타입 (함수 타입 alias) |
| `CatalogEntry<T>` | 타입 (제네릭 유니온 타입 alias) |
| `Catalog` | 인터페이스 및 `InjectionToken<Catalog>` 상수 (동명 선언) |

## 상세 명세

### `CatalogLoader`

```
type CatalogLoader = () =>
  | Promise<Type<DynamicComponent<any>>>
  | Type<DynamicComponent<any>>;
```

- 인자 없는 함수 타입이다.
- 동기적으로 `Type<DynamicComponent<any>>`를 반환하거나, 비동기적으로 동일한 타입을 감싼 `Promise`를 반환한다.
- 지연 로딩(lazy loading)을 위해 Promise 반환 경로를 지원한다.

### `CatalogEntry<T extends AnyComponentNode>`

```
type CatalogEntry<T extends AnyComponentNode> =
  | CatalogLoader
  | {
      type: CatalogLoader;
      bindings: (data: T) => Binding[];
    };
```

- 유니온 타입으로, 단순 `CatalogLoader` 함수이거나 `type`과 `bindings` 두 필드를 가진 객체일 수 있다.
- 객체 형태에서 `bindings` 함수는 컴포넌트 노드 데이터(`data: T`)를 받아 Angular `Binding[]` 배열을 반환한다. 이는 컴포넌트에 추가적인 바인딩을 선언적으로 주입할 때 사용된다.

### `Catalog` (인터페이스)

```
interface Catalog {
  [key: string]: CatalogEntry<AnyComponentNode>;
}
```

- 임의의 문자열 키(컴포넌트 타입 이름)를 `CatalogEntry`에 매핑하는 인덱스 시그니처 인터페이스다.
- 키는 컴포넌트 노드의 `type` 필드 값과 정확히 일치해야 한다.

### `Catalog` (InjectionToken)

```
export const Catalog = new InjectionToken<Catalog>('Catalog');
```

- Angular DI 시스템에서 `Catalog` 인터페이스를 제공·주입하기 위한 토큰이다.
- 인터페이스와 동명으로 선언되어 있어, TypeScript에서 타입 위치와 값 위치 모두에서 `Catalog`를 사용할 수 있다(declaration merging 패턴).
- 토큰 설명 문자열은 `'Catalog'`이다.

## 동작 흐름

이 파일은 선언만 포함하며 런타임 로직은 없다. `Renderer`가 생성될 때 `inject(Catalog)`로 이 토큰에 바인딩된 카탈로그 객체를 주입받는다. 소비자는 Angular 모듈의 `providers` 배열에 `{ provide: Catalog, useValue: myCatalog }` 형태로 실제 카탈로그 객체를 제공해야 한다.
