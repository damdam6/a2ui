# renderers/web_core/src/v0_9/index.ts

## 개요

A2UI v0.9 웹 코어 렌더링 라이브러리의 최상위 공개 진입점(barrel) 파일이다. 데이터 모델, 컴포넌트 모델, 카탈로그 시스템, 이벤트, 스키마, 에러 클래스, 기본 카탈로그 등 v0.9의 모든 공개 모듈을 집약해 재내보내며, Preact Signals의 핵심 유틸리티와 서버→클라이언트 JSON 스키마 원본도 함께 노출한다.

## 의존성

### 외부 패키지
- `@preact/signals-core` — `effect`, `Signal`, `signal`, `computed`

### 저장소 내부 모듈

- [`./catalog/function_invoker.js`](./catalog/function_invoker.ts.md) — `FunctionInvoker` 타입
- [`./catalog/types.js`](./catalog/types.ts.md) — 카탈로그 핵심 타입 및 `Catalog` 클래스
- [`./common/events.js`](./common/events.ts.md) — `EventEmitter`, `EventSource`, `Subscription`
- [`./processing/message-processor.js`](./processing/message-processor.ts.md) — 메시지 처리기
- [`./rendering/component-context.js`](./rendering/component-context.ts.md) — 컴포넌트 컨텍스트
- [`./rendering/data-context.js`](./rendering/data-context.ts.md) — `DataContext`
- [`./rendering/generic-binder.js`](./rendering/generic-binder.ts.md) — 제네릭 바인더
- [`./schema/index.js`](./schema/index.ts.md) — 스키마 정의
- [`./state/component-model.js`](./state/component-model.ts.md) — 컴포넌트 상태 모델
- [`./state/data-model.js`](./state/data-model.ts.md) — 데이터 상태 모델
- [`./state/surface-components-model.js`](./state/surface-components-model.ts.md) — 서피스 컴포넌트 모델
- [`./state/surface-group-model.js`](./state/surface-group-model.ts.md) — 서피스 그룹 모델
- [`./state/surface-model.js`](./state/surface-model.ts.md) — 서피스 모델
- [`./errors.js`](./errors.ts.md) — 에러 클래스
- [`./basic_catalog/index.js`](./basic_catalog/index.ts.md) — 기본 카탈로그 전체
- `./schemas/server_to_client.json` — 서버→클라이언트 메시지 JSON 스키마 원본

## Exports

| 이름/패턴 | 출처 | 설명 |
|---|---|---|
| `export * from './catalog/function_invoker.js'` | function_invoker | `FunctionInvoker` 타입 |
| `export * from './catalog/types.js'` | types | 카탈로그 타입 및 클래스 |
| `export * from './common/events.js'` | events | 이벤트 시스템 |
| `export * from './processing/message-processor.js'` | message-processor | 메시지 처리 |
| `export * from './rendering/component-context.js'` | component-context | 컴포넌트 컨텍스트 |
| `export * from './rendering/data-context.js'` | data-context | 데이터 컨텍스트 |
| `export * from './rendering/generic-binder.js'` | generic-binder | 제네릭 바인더 |
| `export * from './schema/index.js'` | schema | 스키마 정의 |
| `export * from './state/component-model.js'` | component-model | 컴포넌트 모델 |
| `export * from './state/data-model.js'` | data-model | 데이터 모델 |
| `export * from './state/surface-components-model.js'` | surface-components-model | 서피스 컴포넌트 모델 |
| `export * from './state/surface-group-model.js'` | surface-group-model | 서피스 그룹 모델 |
| `export * from './state/surface-model.js'` | surface-model | 서피스 모델 |
| `export * from './errors.js'` | errors | A2UI 에러 클래스 |
| `export * from './basic_catalog/index.js'` | basic_catalog | 기본 카탈로그 전체 |
| `effect`, `Signal`, `signal`, `computed` | `@preact/signals-core` | Preact Signals 핵심 유틸리티 |
| `Schemas` | 이 파일 | `{ A2uiMessageSchemaRaw }` — 서버→클라이언트 JSON 스키마 원본 객체 |

## 상세 명세

### `Schemas` 상수

```ts
export const Schemas = {
  A2uiMessageSchemaRaw,
};
```

`./schemas/server_to_client.json` 파일을 `with { type: 'json' }` 임포트 assertion으로 로드한 원본 객체를 `A2uiMessageSchemaRaw`로 담아 내보낸다. 런타임 스키마 검증이나 문서화 목적으로 소비자가 JSON 스키마 원본에 접근할 때 사용한다.

## 동작 흐름

이 파일은 배럴(barrel) 역할만 하며 자체 로직이 없다. 임포트 시 모든 하위 모듈이 로드되고, 소비자는 이 단일 진입점을 통해 v0.9의 모든 공개 API에 접근한다. JSON 스키마는 `import ... with { type: 'json' }` 구문으로 정적 로드된다.
