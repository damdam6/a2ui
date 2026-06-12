# renderers/web_core/src/v0_9/schema/client-to-server.ts

## 개요

A2UI 클라이언트가 서버로 전송하는 모든 메시지 구조를 Zod 스키마로 정의한다. 사용자 인터랙션으로 발생한 액션 메시지(`A2uiClientActionSchema`)와 클라이언트 측 오류 보고 메시지(`A2uiClientErrorSchema`)를 단일 유니온 스키마(`A2uiClientMessageSchema`)로 묶고, 데이터 모델 동기화 스키마(`A2uiClientDataModelSchema`)도 함께 정의한다. 사양 파일 `specification/v0_9/json/client_to_server.json` 및 `client_data_model.json`에 대응한다.

## 의존성

### 외부 패키지
- `zod` (`z`)

### 저장소 내부 모듈

없음.

## Exports

| 이름 | 종류 | 설명 |
|------|------|------|
| `A2uiClientActionSchema` | Zod 스키마 상수 | 사용자 인터랙션 액션 메시지 스키마 |
| `A2uiValidationErrorSchema` | Zod 스키마 상수 | 클라이언트 측 유효성 검사 오류 스키마 |
| `A2uiGenericErrorSchema` | Zod 스키마 상수 | 일반 클라이언트 오류 스키마 |
| `A2uiClientErrorSchema` | Zod 스키마 상수 | 유효성 검사 오류와 일반 오류의 유니온 스키마 |
| `A2uiClientMessageSchema` | Zod 스키마 상수 | 클라이언트→서버 메시지 최상위 스키마 |
| `A2uiClientDataModelSchema` | Zod 스키마 상수 | 데이터 모델 동기화 메시지 스키마 |
| `A2uiClientAction` | 타입 | `A2uiClientActionSchema` 추론 타입 |
| `A2uiClientError` | 타입 | `A2uiClientErrorSchema` 추론 타입 |
| `A2uiClientMessage` | 타입 | `A2uiClientMessageSchema` 추론 타입 |
| `A2uiClientDataModel` | 타입 | `A2uiClientDataModelSchema` 추론 타입 |
| `A2uiClientMessageListSchema` | Zod 스키마 상수 | 클라이언트 메시지 배열 스키마 |
| `A2uiClientMessageList` | 타입 | `A2uiClientMessageListSchema` 추론 타입 |
| `A2uiClientMessageListWrapperSchema` | Zod 스키마 상수 | `{ messages: [...] }` 래퍼 스키마 |
| `A2uiClientMessageListWrapper` | 타입 | `A2uiClientMessageListWrapperSchema` 추론 타입 |

## 상세 명세

### 상수: `A2uiClientActionSchema`

`z.object({ ... }).strict()`로 정의된 스키마. 사양 `client_to_server.json`의 `action` 필드에 대응한다.

- `name: z.string()` — 컴포넌트 `action.event.name` 프로퍼티에서 가져온 액션 이름.
- `surfaceId: z.string()` — 이벤트가 발생한 surface ID.
- `sourceComponentId: z.string()` — 이벤트를 발생시킨 컴포넌트 ID.
- `timestamp: z.string().datetime()` — ISO 8601 형식의 이벤트 발생 시각.
- `context: z.record(z.any())` — 데이터 바인딩이 모두 해석된 후의 `action.event.context` 키-값 쌍.

스키마에 `.strict()`가 적용되어 미지정 키가 있으면 파싱에 실패한다.

### 상수: `A2uiValidationErrorSchema`

`z.object({ ... }).strict()`로 정의된 스키마. 클라이언트 측 유효성 검사 실패를 보고한다.

- `code: z.literal('VALIDATION_FAILED')` — 고정 리터럴 값.
- `surfaceId: z.string()` — 오류 발생 surface ID.
- `path: z.string()` — 유효성 검사에 실패한 필드의 JSON Pointer (예: `'/components/0/text'`).
- `message: z.string()` — 유효성 검사 실패 이유를 1~2문장으로 설명.

### 상수: `A2uiGenericErrorSchema`

`z.object({ ... }).passthrough()`로 정의된 스키마. `VALIDATION_FAILED` 이외의 모든 오류를 처리한다.

- `code: z.string().refine(c => c !== 'VALIDATION_FAILED')` — `'VALIDATION_FAILED'`가 아닌 임의 문자열. `refine`으로 런타임 검증.
- `message: z.string()` — 오류 이유.
- `surfaceId: z.string()` — 오류 발생 surface ID.

`.passthrough()`가 적용되어 추가 키를 허용한다.

### 상수: `A2uiClientErrorSchema`

`z.union([A2uiValidationErrorSchema, A2uiGenericErrorSchema])`. Zod 유니온은 순서대로 시도하므로 `VALIDATION_FAILED` 코드를 가진 객체는 `A2uiValidationErrorSchema`로, 그 외는 `A2uiGenericErrorSchema`로 파싱된다.

### 상수: `A2uiClientMessageSchema`

`z.object({ version: z.literal('v0.9') }).and(z.union([...]))` 형태의 교차(intersection) 스키마.

- `version: z.literal('v0.9')` — 반드시 문자열 `'v0.9'`여야 한다.
- `.and(z.union([z.object({action: A2uiClientActionSchema}), z.object({error: A2uiClientErrorSchema})]))` — `action` 또는 `error` 중 하나를 포함해야 한다.

### 상수: `A2uiClientDataModelSchema`

`z.object({ ... }).strict()` 스키마. `client_data_model.json`에 대응한다.

- `version: z.literal('v0.9')` — 반드시 `'v0.9'`여야 한다.
- `surfaces: z.record(z.object({}).passthrough())` — surface ID를 키로, 임의의 데이터 모델 객체를 값으로 가지는 맵.

### 상수: `A2uiClientMessageListSchema`

`z.array(A2uiClientMessageSchema)`. 클라이언트 메시지의 배열을 표현한다.

### 상수: `A2uiClientMessageListWrapperSchema`

`z.object({ messages: A2uiClientMessageListSchema }).strict()`. 클라이언트 메시지 목록을 `messages` 키로 래핑하는 스키마.

## 동작 흐름

이 파일은 런타임 로직 없이 스키마 정의와 타입 추론 선언만 포함한다. 클라이언트 런타임이 서버로 메시지를 전송하기 전 또는 서버가 수신 데이터를 검증할 때 `safeParse` 또는 `parse`를 호출하여 사용한다.
