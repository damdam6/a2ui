# renderers/web_core/src/v0_9/schema/client-capabilities.ts

## 개요

클라이언트가 서버에 전송하는 능력(capabilities) 메타데이터 구조를 TypeScript 인터페이스로 정의한다. 지원하는 카탈로그 ID 목록과 인라인 카탈로그 정의를 포함하며, 이를 통해 서버는 클라이언트가 렌더링할 수 있는 컴포넌트 세트와 함수, 테마를 파악한다. 이 파일은 Zod를 사용하지 않는 순수 TypeScript 인터페이스 파일로, 런타임 검증 없이 타입 안전성만 제공한다.

## 의존성

외부 패키지 및 내부 모듈 import 없음.

## Exports

| 이름 | 종류 | 설명 |
|------|------|------|
| `JsonSchema` | 타입 | JSON Schema 정의를 나타내는 `Record<string, any>` 별칭 |
| `FunctionDefinition` | 인터페이스 | 인라인 카탈로그 내 함수의 인터페이스 |
| `InlineCatalog` | 인터페이스 | 인라인 카탈로그 정의 구조 |
| `A2uiClientCapabilities` | 인터페이스 | 클라이언트가 서버에 보내는 최상위 능력 구조 |

## 상세 명세

### 타입: `JsonSchema`

`Record<string, any>` 의 타입 별칭. 무거운 JSON Schema 타입 라이브러리 없이 표준 JSON Schema 프로퍼티를 허용하기 위해 사용한다.

### 인터페이스: `FunctionDefinition`

인라인 카탈로그 내 함수의 인터페이스를 기술한다.

- `name: string` — 함수 이름.
- `description?: string` — 선택적 설명.
- `parameters: JsonSchema` — 함수 매개변수 JSON Schema.
- `returnType: 'string' | 'number' | 'boolean' | 'array' | 'object' | 'any' | 'void'` — 반환 타입 리터럴 유니온. 총 7가지 값 중 하나여야 한다.

### 인터페이스: `InlineCatalog`

`A2uiClientCapabilities` 객체 내에서 카탈로그를 인라인으로 정의하는 구조체.

- `catalogId: string` — 카탈로그를 고유하게 식별하는 문자열.
- `components?: Record<string, JsonSchema>` — 선택적. 컴포넌트 이름을 키로, JSON Schema를 값으로 하는 맵.
- `functions?: FunctionDefinition[]` — 선택적. 함수 정의 배열.
- `theme?: Record<string, JsonSchema>` — 선택적. 테마 파라미터를 키-JSON Schema 쌍으로 표현하는 맵.

### 인터페이스: `A2uiClientCapabilities`

클라이언트가 서버로 전송하는 최상위 능력 구조체.

- `'v0.9': { supportedCatalogIds: string[]; inlineCatalogs?: InlineCatalog[]; }` — 버전 키 `'v0.9'` 아래에 두 필드를 가진 객체가 위치한다.
  - `supportedCatalogIds: string[]` — 클라이언트가 지원하는 카탈로그 ID 목록.
  - `inlineCatalogs?: InlineCatalog[]` — 선택적. 전송에 인라인으로 포함할 카탈로그 정의 배열.

## 동작 흐름

이 파일은 런타임 로직 없이 타입 정의만 포함한다. 클라이언트는 `A2uiClientCapabilities` 타입의 객체를 생성해 트랜스포트 메타데이터로 서버에 전달하며, 서버는 이를 통해 클라이언트의 렌더링 능력을 파악한다.
