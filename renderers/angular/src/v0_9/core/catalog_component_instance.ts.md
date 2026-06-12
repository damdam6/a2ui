# renderers/angular/src/v0_9/core/catalog_component_instance.ts

## 개요

카탈로그 컴포넌트가 반드시 구현해야 하는 최소 계약을 정의하는 인터페이스 파일이다. Angular `Signal` 타입을 사용하여 네 개의 읽기 전용 signal 필드를 선언하며, 이를 통해 `ComponentHostComponent`가 임의의 컴포넌트 타입에 대해 타입 안전하게 props와 컨텍스트 정보를 전달할 수 있다.

## 의존성

### 외부 패키지
- `@angular/core`: `Signal`

### 저장소 내부 모듈
없음.

## Exports

| 이름 | 종류 |
|------|------|
| `CatalogComponentInstance` | 인터페이스 |

## 상세 명세

### `CatalogComponentInstance`

인터페이스. 카탈로그 컴포넌트가 구현해야 하는 signal 기반 속성 계약을 정의한다.

| 필드 | 타입 | 설명 |
|------|------|------|
| `props` (readonly) | `Signal<Record<string, unknown>>` | 컴포넌트의 바인딩된 프로퍼티 맵 |
| `surfaceId` (readonly) | `Signal<string>` | 컴포넌트가 속한 서피스 식별자 |
| `componentId` (readonly) | `Signal<string>` | 컴포넌트의 고유 식별자 |
| `dataContextPath` (readonly) | `Signal<string>` | 데이터 컨텍스트 기반 경로 |

## 동작 흐름

이 파일은 런타임 로직 없이 인터페이스만 선언한다. `AngularComponentImplementation`의 `component` 필드 타입(`Type<CatalogComponentInstance>`)으로 참조되어, Angular DI가 동적으로 컴포넌트를 인스턴스화할 때 필요한 입력 속성의 존재를 컴파일 타임에 보장한다.
