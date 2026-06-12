# renderers/web_core/src/v0_9/catalog/types.ts

## 개요

A2UI 카탈로그 시스템의 핵심 타입과 `Catalog` 클래스를 정의하는 파일이다. 함수 API 명세(`FunctionApi`), 함수 구현체(`FunctionImplementation`), 컴포넌트 API 명세(`ComponentApi`), 그리고 이 둘을 등록·관리하는 `Catalog` 클래스를 제공한다. `Catalog`는 함수 호출 시 Zod 스키마로 인수를 검증하고, 오류 발생 시 `A2uiExpressionError`를 던지는 `invoker` 콜백을 자체적으로 생성한다.

## 의존성

### 외부 패키지
- `zod` — 스키마 정의 및 런타임 검증
- `@preact/signals-core` — `Signal` 타입

### 저장소 내부 모듈

- [`../rendering/data-context.js`](../rendering/data-context.ts.md) — `DataContext` 타입
- [`../errors.js`](../errors.ts.md) — `A2uiExpressionError`
- [`./function_invoker.js`](./function_invoker.ts.md) — `FunctionInvoker` 타입

## Exports

| 이름 | 종류 | 설명 |
|---|---|---|
| `isSignal` | 함수 | 값이 Preact Signal인지 패키지 경계 안전하게 판별 |
| `A2uiReturnType` | 타입(union) | 지원하는 반환 타입 문자열 리터럴 집합 |
| `InferA2uiReturnType` | 타입(조건부 제네릭) | `A2uiReturnType` 문자열을 실제 TypeScript 타입으로 매핑 |
| `FunctionApi` | 인터페이스 | 함수의 API 계약 (name, returnType, schema) |
| `FunctionImplementation` | 인터페이스 | 실행 가능한 함수 구현체 (`FunctionApi` + `execute`) |
| `createFunctionImplementation` | 함수 | 타입 안전한 `FunctionImplementation` 생성 팩토리 |
| `ComponentApi` | 인터페이스 | 컴포넌트의 API 계약 (name, schema) |
| `InferredComponentApiSchemaType` | 타입(유틸리티) | `ComponentApi`의 스키마 타입을 추론 |
| `CatalogInterface` | 인터페이스(declare) | `Catalog`의 공개 속성 선언 (Closure Compiler 대응) |
| `Catalog` | 클래스 | 컴포넌트·함수를 등록·관리하고 invoker를 제공하는 카탈로그 |

## 상세 명세

### `isSignal(val: any): val is Signal<any>`

- 매개변수: `val` — 검사할 임의 값
- 반환: `val is Signal<any>` (타입 가드)
- `val`이 truthy이고, `typeof val === 'object'`이며, `'value'` 프로퍼티와 `'peek'` 프로퍼티를 모두 갖고 있으면 `true`를 반환한다. 패키지 경계를 넘어도 작동하도록 `instanceof` 대신 덕 타이핑 방식을 사용한다.

---

### `A2uiReturnType`

`'string' | 'number' | 'boolean' | 'array' | 'object' | 'any' | 'void'` 중 하나의 문자열 리터럴 유니언 타입.

---

### `InferA2uiReturnType<T extends A2uiReturnType>`

조건부 타입. `T`에 따라 매핑:
- `'string'` → `string`
- `'number'` → `number`
- `'boolean'` → `boolean`
- `'array'` → `any[]`
- `'object'` → `Record<string, any>`
- `'void'` → `void`
- 그 외(`'any'` 포함) → `any`

---

### 인터페이스: `FunctionApi`

| 필드 | 타입 | 설명 |
|---|---|---|
| `name` | `readonly string` | 함수 이름 |
| `returnType` | `readonly A2uiReturnType` | 반환 타입 문자열 |
| `schema` | `readonly z.ZodTypeAny` | 인수 검증 Zod 스키마 |

---

### 인터페이스: `FunctionImplementation extends FunctionApi`

`FunctionApi`를 확장하며 `execute` 메서드를 추가한다.

`execute(args: Record<string, any>, context: DataContext, abortSignal?: AbortSignal): unknown | Signal<unknown>`

---

### `createFunctionImplementation<Schema extends z.ZodTypeAny, TReturn extends A2uiReturnType>`

- **매개변수**:
  - `api`: `{ name: string; returnType: TReturn; schema: Schema }` — API 디스크립터
  - `execute`: `(args: z.infer<Schema>, context: DataContext, abortSignal?: AbortSignal) => InferA2uiReturnType<TReturn> | Signal<InferA2uiReturnType<TReturn>>` — 타입 안전한 실행 함수
- **반환**: `FunctionImplementation`

`api`의 `name`, `returnType`, `schema`를 그대로 복사하고, `execute`를 내부 타입 캐스팅(`as`)으로 `FunctionImplementation.execute` 시그니처에 맞춰 반환한다. 제네릭을 통해 `execute`의 `args` 타입이 스키마에서 자동 추론된다.

---

### 인터페이스: `ComponentApi<Schema extends z.ZodTypeAny = z.ZodTypeAny>`

| 필드 | 타입 | 설명 |
|---|---|---|
| `name` | `string` | A2UI JSON 내 컴포넌트 이름 (예: `'Button'`) |
| `schema` | `readonly Schema` | 컴포넌트 프로퍼티 Zod 스키마. `'component'`와 `'id'` 필드는 포함 금지 |

---

### `InferredComponentApiSchemaType<Api extends ComponentApi>`

`z.infer<Api['schema']>` 의 별칭. `ComponentApi`를 `satisfies` 키워드로 좁혀서 사용해야 `any`로 퇴화하지 않고 정확한 타입이 추론된다.

---

### `declare interface CatalogInterface<T extends ComponentApi>`

Closure Compiler에서 프로퍼티 이름 단축(renaming)을 방지하기 위한 인터페이스 선언. `Catalog`의 모든 공개 프로퍼티를 명시한다:

| 필드 | 타입 |
|---|---|
| `id` | `readonly string` |
| `components` | `ReadonlyMap<string, T>` |
| `functions` | `ReadonlyMap<string, FunctionImplementation>` |
| `themeSchema` | `z.ZodObject<any>` (optional) |
| `invoker` | `FunctionInvoker` |

---

### `class Catalog<T extends ComponentApi> implements CatalogInterface<T>`

#### 필드

| 필드 | 타입 | 설명 |
|---|---|---|
| `id` | `readonly string` | 카탈로그 식별자 |
| `components` | `ReadonlyMap<string, T>` | 컴포넌트 이름→`ComponentApi` 매핑 |
| `functions` | `ReadonlyMap<string, FunctionImplementation>` | 함수 이름→`FunctionImplementation` 매핑 |
| `themeSchema` | `z.ZodObject<any>` (optional) | 테마 파라미터 스키마 |
| `invoker` | `readonly FunctionInvoker` | 함수 호출 콜백 (생성자에서 할당) |

#### `constructor(id: string, components: T[], functions: FunctionImplementation[] = [], themeSchema?: z.ZodObject<any>)`

1. `id` 저장
2. `components` 배열을 순회해 `comp.name`을 키로 `Map<string, T>` 생성 → `this.components`
3. `functions` 배열을 순회해 `fn.name`을 키로 `Map<string, FunctionImplementation>` 생성 → `this.functions`
4. `themeSchema` 저장 (옵션)
5. `this.invoker`에 클로저 함수 할당:
   - `this.functions.get(name)`으로 함수 조회. 없으면 `A2uiExpressionError("Function not found in catalog '${this.id}': ${name}", name)` 던짐
   - `fn.schema.parse(rawArgs)`로 Zod 검증 및 강제 변환 수행. `ZodError` 발생 시 `A2uiExpressionError("Validation failed for function '${name}': ...", name, e.errors ?? e.issues)` 던짐
   - 검증 성공 시 `fn.execute(safeArgs, ctx, abortSignal)` 결과 반환
   - 그 외 예외는 재던짐(rethrow)

## 동작 흐름

`Catalog` 인스턴스 생성 시 컴포넌트 배열과 함수 배열이 각각 `ReadonlyMap`으로 인덱싱된다. 이후 렌더러나 표현식 평가기가 `catalog.invoker`를 `DataContext`에 전달하면, 표현식 파서가 함수 호출을 만날 때마다 해당 콜백을 통해 함수를 이름으로 조회·검증·실행한다.
