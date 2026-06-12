# renderers/web_core/src/v0_9/schema/common-types.ts

## 개요

A2UI v0.9 프로토콜 전반에서 공유되는 핵심 Zod 스키마와 TypeScript 타입을 정의한다. 데이터 바인딩(`DataBinding`), 함수 호출(`FunctionCall`), 동적 값 타입 패밀리(`DynamicString`, `DynamicNumber`, `DynamicBoolean`, `DynamicStringList`, `DynamicValue`), 컴포넌트 식별자, 자식 목록(`ChildList`), 액션(`Action`), 유효성 검사 규칙(`CheckRule`, `Checkable`), 접근성 속성(`AccessibilityAttributes`), 범용 컴포넌트 스키마(`AnyComponent`)를 포함한다. `generic-binder.ts`의 스키마 스크래핑 로직은 이 파일에서 정의된 union 형태를 기반으로 동작 유형을 추론한다.

## 의존성

### 외부 패키지
- `zod` (`z`)

### 저장소 내부 모듈

없음.

## Exports

| 이름 | 종류 | 설명 |
|------|------|------|
| `DataBindingSchema` | Zod 스키마 상수 | 데이터 모델 경로 바인딩 스키마 |
| `DataBindingType` / `DataBinding` | 타입 | `DataBindingSchema` 추론 타입 |
| `FunctionCallSchema` | Zod 스키마 상수 | 클라이언트 함수 호출 스키마 |
| `FunctionCallType` / `FunctionCall` | 타입 | `FunctionCallSchema` 추론 타입 |
| `DynamicBooleanSchema` | Zod 스키마 상수 | 동적 불린 스키마 |
| `DynamicStringSchema` | Zod 스키마 상수 | 동적 문자열 스키마 |
| `DynamicNumberSchema` | Zod 스키마 상수 | 동적 숫자 스키마 |
| `DynamicStringListSchema` | Zod 스키마 상수 | 동적 문자열 배열 스키마 |
| `DynamicValueSchema` | Zod 스키마 상수 | 임의 타입의 동적 값 스키마 |
| `DynamicBoolean` / `DynamicString` / `DynamicNumber` / `DynamicStringList` / `DynamicValue` | 타입 | 각 동적 스키마의 추론 타입 |
| `ComponentIdSchema` | Zod 스키마 상수 | 컴포넌트 고유 식별자 스키마 |
| `ComponentId` | 타입 | `ComponentIdSchema` 추론 타입 |
| `ChildListSchema` | Zod 스키마 상수 | 정적 또는 동적 자식 목록 스키마 |
| `ChildList` | 타입 | `ChildListSchema` 추론 타입 |
| `ActionSchema` | Zod 스키마 상수 | 서버 이벤트 또는 클라이언트 함수 실행 액션 스키마 |
| `Action` | 타입 | `ActionSchema` 추론 타입 |
| `CheckRuleSchema` | Zod 스키마 상수 | 유효성 검사 규칙 스키마 |
| `CheckRule` | 타입 | `CheckRuleSchema` 추론 타입 |
| `CheckableSchema` | Zod 스키마 상수 | `checks` 배열을 가진 객체 스키마 |
| `Checkable` | 타입 | `CheckableSchema` 추론 타입 |
| `AccessibilityAttributesSchema` | Zod 스키마 상수 | 접근성 속성 스키마 |
| `AccessibilityAttributes` | 타입 | `AccessibilityAttributesSchema` 추론 타입 |
| `AnyComponentSchema` | Zod 스키마 상수 | 범용 A2UI 컴포넌트 스키마 |
| `AnyComponent` | 타입 | `AnyComponentSchema` 추론 타입 |
| `CommonSchemas` | 상수 (객체) | 위 모든 스키마를 이름을 키로 모은 레지스트리 객체 |

## 상세 명세

### 상수: `DataBindingSchema`

`z.object({ path: z.string() })`. `path`는 데이터 모델 내 값을 가리키는 JSON Pointer 문자열이다. describe 주석에 `REF:common_types.json#/$defs/DataBinding`을 포함해 JSON Schema 사양과 연결된다.

### 상수: `FunctionCallSchema`

`z.object({ call: z.string(), args: z.record(z.any()), returnType: z.enum([...]).default('boolean') })`. `call`은 호출할 함수 이름, `args`는 임의 키-값 인수, `returnType`은 `'string' | 'number' | 'boolean' | 'array' | 'object' | 'any' | 'void'` 중 하나이며 기본값은 `'boolean'`이다.

### 동적 값 스키마 패밀리

모두 `z.union([<literal 타입>, DataBindingSchema, FunctionCallSchema])` 형태의 3-way 유니온이다.

- **`DynamicBooleanSchema`**: `[z.boolean(), DataBindingSchema, FunctionCallSchema]`
- **`DynamicStringSchema`**: `[z.string(), DataBindingSchema, FunctionCallSchema]`
- **`DynamicNumberSchema`**: `[z.number(), DataBindingSchema, FunctionCallSchema]`
- **`DynamicStringListSchema`**: `[z.array(z.string()), DataBindingSchema, FunctionCallSchema]`
- **`DynamicValueSchema`**: `[z.string(), z.number(), z.boolean(), z.array(z.any()), DataBindingSchema, FunctionCallSchema]` — 6-way 유니온으로 모든 원시 타입을 포함한다.

`generic-binder.ts`의 `getFieldBehavior`는 이 union들을 검사하여 `DYNAMIC` 동작을 식별한다: `DataBindingSchema`가 `{ path }` 형태이고 `{ componentId }`가 없는 경우.

### 상수: `ComponentIdSchema`

`z.string()`. 컴포넌트 고유 식별자를 나타내는 단순 문자열 스키마다.

### 상수: `ChildListSchema`

`z.union([z.array(ComponentIdSchema), z.object({ componentId: ComponentIdSchema, path: z.string() })])`. 두 가지 형태를 허용한다:
- **정적 목록**: 컴포넌트 ID 문자열 배열.
- **동적 템플릿**: `componentId`(반복할 컴포넌트 ID)와 `path`(데이터 모델 내 목록 경로)를 가진 객체.

`generic-binder.ts`는 `{ componentId, path }` 형태를 감지하여 `STRUCTURAL` 동작으로 분류한다.

### 상수: `ActionSchema`

`z.union([z.object({ event: z.object({ name: z.string(), context: z.record(DynamicValueSchema).optional() }) }), z.object({ functionCall: FunctionCallSchema })])`. 두 가지 형태를 허용한다:
- `{ event: { name, context? } }` — 서버 측 이벤트를 트리거한다.
- `{ functionCall: FunctionCallSchema }` — 로컬 클라이언트 함수를 실행한다.

`generic-binder.ts`는 `{ event: ... }`를 포함한 union을 `ACTION` 동작으로 분류한다.

### 상수: `CheckRuleSchema`

`z.object({ condition: DynamicBooleanSchema, message: z.string() })`. 유효성 검사 규칙 하나를 표현한다. `condition`은 동적 불린 값이고, `message`는 실패 시 표시할 오류 메시지다.

### 상수: `CheckableSchema`

`z.object({ checks: z.array(CheckRuleSchema).optional() })`. `checks` 키를 선택적으로 가진 객체 스키마다. 이 스키마를 포함하는 컴포넌트는 `generic-binder.ts`에서 `CHECKABLE` 동작으로 처리된다.

### 상수: `AccessibilityAttributesSchema`

`z.object({ label: DynamicStringSchema.optional(), description: DynamicStringSchema.optional() })`. 보조 기술을 위한 `label`과 `description`을 선택적으로 제공한다.

### 상수: `AnyComponentSchema`

`z.object({ component: z.string(), id: ComponentIdSchema.optional(), weight: z.number().optional() }).passthrough()`. 모든 A2UI 컴포넌트의 최소 공통 구조. `component` 필드(컴포넌트 타입명)가 필수이며, `.passthrough()`로 추가 프로퍼티를 허용한다.

### 상수: `CommonSchemas`

모든 스키마를 의미 있는 이름(`ComponentId`, `ChildList`, `DataBinding`, `DynamicValue`, `DynamicString`, `DynamicNumber`, `DynamicBoolean`, `DynamicStringList`, `FunctionCall`, `CheckRule`, `Checkable`, `Action`, `AccessibilityAttributes`, `AnyComponent`)을 키로 모은 레지스트리 객체다. 외부에서 스키마를 동적으로 조회할 때 사용한다.

## 동작 흐름

이 파일은 런타임 로직 없이 스키마 정의와 타입 선언만 포함한다. 다른 모듈들은 이 파일에서 내보내는 스키마를 import하여 Zod 파싱, 타입 검사, 그리고 `generic-binder.ts`의 스키마 스크래핑에 활용한다.
