# renderers/angular/src/v0_9/core/catalog_component.ts

## 개요

모든 Angular A2UI 카탈로그 컴포넌트의 추상 기반 지시자(Directive)다. `CatalogComponentInstance` 인터페이스를 구현하며, `props`, `surfaceId`, `componentId`, `dataContextPath` 네 개의 Angular signal input을 공통으로 제공한다. 제네릭 타입 파라미터 `Api`를 통해 컴포넌트별 props 타입 안전성을 보장한다.

## 의존성

### 외부 패키지
- `@angular/core`: `Directive`, `input`
- `@a2ui/web_core/v0_9`: `ComponentApi`

### 저장소 내부 모듈
- [`./types`](./types.ts.md) — `ComponentApiToProps`
- [`./catalog_component_instance`](./catalog_component_instance.ts.md)

## Exports

| 이름 | 종류 |
|------|------|
| `CatalogComponent` | 추상 Angular Directive 클래스 (제네릭) |

## 상세 명세

### `CatalogComponent<Api extends ComponentApi>`

**데코레이터**: `@Directive()`  
**구현 인터페이스**: `CatalogComponentInstance`  
**제네릭**: `Api extends ComponentApi` — 각 구체 컴포넌트가 사용하는 API 타입 (예: `typeof VideoApi`)

#### 필드

| 이름 | 타입 | 종류 | 기본값 | 설명 |
|------|------|------|--------|------|
| `props` | `Signal<ComponentApiToProps<Api>>` | `input<ComponentApiToProps<Api>>` | `{} as ComponentApiToProps<Api>` | A2UI ComponentModel에서 resolve된 reactive 프로퍼티 맵 |
| `surfaceId` | `Signal<string>` | `input.required<string>()` | — | 이 컴포넌트가 속한 서피스 ID |
| `componentId` | `Signal<string>` | `input.required<string>()` | — | 컴포넌트의 고유 식별자 |
| `dataContextPath` | `Signal<string>` | `input<string>` | `'/'` | 데이터 컨텍스트 경로. 기본값은 루트 `'/'`. |

`props` 입력의 기본값은 빈 객체 `{}`를 `ComponentApiToProps<Api>` 타입으로 단언(cast)한 것이다. `surfaceId`와 `componentId`는 required input이므로 부모 컴포넌트에서 반드시 전달해야 한다.

## 동작 흐름

이 클래스는 추상 클래스이므로 직접 인스턴스화되지 않는다. 구체 컴포넌트(예: `VideoComponent`)가 `extends CatalogComponent<typeof SomeApi>`로 상속하여 공통 입력 signal 네 개를 그대로 사용하고, 추가적인 `computed` signal(예: `url`)을 선언하여 `props()`에서 값을 파생시킨다.
