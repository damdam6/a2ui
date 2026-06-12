# renderers/web_core/src/v0_9/schema/server-to-client.ts

## 개요

A2UI 서버가 클라이언트로 전송하는 모든 메시지 구조를 Zod 스키마와 TypeScript 인터페이스로 이중으로 정의한다. surface 생성(`createSurface`), 컴포넌트 업데이트(`updateComponents`), 데이터 모델 업데이트(`updateDataModel`), surface 삭제(`deleteSurface`) 네 가지 메시지 유형을 지원하며, 이를 단일 유니온 스키마 `A2uiMessageSchema`로 묶는다. 사양 파일 `specification/v0_9/json/server_to_client.json`에 대응한다.

## 의존성

### 외부 패키지
- `zod` (`z`)

### 저장소 내부 모듈
- [`./common-types.js`](./common-types.ts.md) — `AnyComponentSchema`

## Exports

| 이름 | 종류 | 설명 |
|------|------|------|
| `CreateSurfaceMessageSchema` | Zod 스키마 상수 | surface 생성 메시지 스키마 |
| `UpdateComponentsMessageSchema` | Zod 스키마 상수 | 컴포넌트 업데이트 메시지 스키마 |
| `UpdateDataModelMessageSchema` | Zod 스키마 상수 | 데이터 모델 업데이트 메시지 스키마 |
| `DeleteSurfaceMessageSchema` | Zod 스키마 상수 | surface 삭제 메시지 스키마 |
| `CreateSurfaceMessage` | 인터페이스 | `CreateSurfaceMessageSchema` 추론 타입 선언 |
| `UpdateComponentsMessage` | 인터페이스 | `UpdateComponentsMessageSchema` 추론 타입 선언 |
| `UpdateDataModelMessage` | 인터페이스 | `UpdateDataModelMessageSchema` 추론 타입 선언 |
| `DeleteSurfaceMessage` | 인터페이스 | `DeleteSurfaceMessageSchema` 추론 타입 선언 |
| `A2uiMessageSchema` | Zod 스키마 상수 | 4가지 메시지의 유니온 스키마 |
| `A2uiMessage` | 타입 | 4가지 메시지 인터페이스의 유니온 타입 |
| `A2uiMessageListSchema` | Zod 스키마 상수 | 메시지 배열 스키마 |
| `A2uiMessageList` | 타입 | `A2uiMessageListSchema` 추론 타입 |
| `A2uiMessageListWrapperSchema` | Zod 스키마 상수 | `{ messages: [...] }` 래퍼 스키마 |
| `A2uiMessageListWrapper` | 타입 | `A2uiMessageListWrapperSchema` 추론 타입 |

## 상세 명세

### 상수: `CreateSurfaceMessageSchema`

`z.object({ version: z.literal('v0.9'), createSurface: z.object({ ... }).strict() }).strict()`. 중첩 객체 필드:

- `surfaceId: z.string()` — 렌더링할 UI surface의 고유 식별자.
- `catalogId: z.string()` — 해당 surface에서 사용할 카탈로그를 고유하게 식별하는 문자열.
- `theme: z.any().optional()` — 선택적. surface 테마 파라미터.
- `sendDataModel: z.boolean().optional()` — 선택적. `true`이면 클라이언트가 전체 데이터 모델을 서버로 전송한다.

`version` 포함 모든 레벨에 `.strict()`가 적용된다.

### 상수: `UpdateComponentsMessageSchema`

`z.object({ version: z.literal('v0.9'), updateComponents: z.object({ ... }).strict() }).strict()`. 중첩 객체 필드:

- `surfaceId: z.string()` — 업데이트할 surface 식별자.
- `components: z.array(AnyComponentSchema).min(1)` — surface의 모든 UI 컴포넌트 목록. 최소 1개 이상이어야 한다.

### 상수: `UpdateDataModelMessageSchema`

`z.object({ version: z.literal('v0.9'), updateDataModel: z.object({ ... }).strict() }).strict()`. 중첩 객체 필드:

- `surfaceId: z.string()` — 업데이트 대상 surface 식별자.
- `path: z.string().optional()` — 선택적. 데이터 모델 내 위치를 가리키는 경로.
- `value: z.any().optional()` — 선택적. 업데이트할 데이터.

### 상수: `DeleteSurfaceMessageSchema`

`z.object({ version: z.literal('v0.9'), deleteSurface: z.object({ surfaceId: z.string() }).strict() }).strict()`. 삭제할 surface의 `surfaceId` 하나만 필요하다.

### 인터페이스: `CreateSurfaceMessage`, `UpdateComponentsMessage`, `UpdateDataModelMessage`, `DeleteSurfaceMessage`

각각 해당 Zod 스키마의 `z.infer<...>`를 `extends`하는 `declare interface`로 선언된다. 이 패턴은 Zod 추론 타입 위에 명시적 인터페이스를 덧씌워 IDE 툴팁 가독성을 높인다. 필드 타입은 스키마와 동일하다.

### 상수: `A2uiMessageSchema`

`z.union([CreateSurfaceMessageSchema, UpdateComponentsMessageSchema, UpdateDataModelMessageSchema, DeleteSurfaceMessageSchema])`. 수신된 메시지를 4가지 타입 중 하나로 파싱하는 최상위 유니온.

### 타입: `A2uiMessage`

`CreateSurfaceMessage | UpdateComponentsMessage | UpdateDataModelMessage | DeleteSurfaceMessage` 유니온 타입.

### 상수: `A2uiMessageListSchema`

`z.array(A2uiMessageSchema)`. 메시지 배열을 표현한다.

### 상수: `A2uiMessageListWrapperSchema`

`z.object({ messages: A2uiMessageListSchema }).strict()`. 메시지 목록을 `messages` 키로 래핑한다.

## 동작 흐름

런타임 로직 없이 스키마 정의와 타입 선언만 포함한다. 서버는 이 스키마 구조에 맞춰 JSON 메시지를 직렬화하고, 클라이언트 런타임은 수신한 메시지를 `A2uiMessageSchema.parse(...)` 또는 `safeParse(...)`로 검증·파싱하여 적절한 처리 분기로 전달한다.
