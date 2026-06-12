# renderers/web_core/src/v0_9/catalog/types.test.ts

## 개요

`catalog/types.ts`에 정의된 `Catalog` 클래스와 `InferredComponentApiSchemaType` 타입의 동작을 검증하는 단위 테스트 파일이다. Node.js 내장 테스트 러너(`node:test`)와 `node:assert`를 사용한다. 런타임 동작과 타입 수준 정확성을 모두 검증한다.

## 의존성

### 외부 패키지
- `zod` — 스키마 정의

### 저장소 내부 모듈

- [`./types.js`](./types.ts.md) — `Catalog`, `ComponentApi`, `InferredComponentApiSchemaType`, `createFunctionImplementation`
- [`../errors.js`](../errors.ts.md) — `A2uiExpressionError`

## 테스트 케이스 명세

### `describe('Catalog Types')`

#### `it('creates a catalog with functions')`

**검증 동작**: `Catalog` 생성자가 컴포넌트와 함수를 올바르게 등록하는지 확인한다.

**픽스처/모킹**:
- `mockComponent`: `{ name: 'MockComp', schema: z.object({}) }` — `ComponentApi`를 만족하는 최소 컴포넌트
- `mockFunc`: `createFunctionImplementation({ name: 'mockFunc', returnType: 'string', schema: z.object({}) }, () => 'result')` — 빈 스키마를 가진 함수 구현체
- `catalog`: `new Catalog('test-cat', [mockComponent], [mockFunc])`

**검증 항목**:
1. `catalog.id === 'test-cat'`
2. `catalog.components.size === 1`
3. `catalog.components.get('MockComp') === mockComponent` — 컴포넌트가 이름 키로 Map에 저장됨
4. `catalog.functions`가 truthy
5. `catalog.functions.size === 1`
6. `catalog.functions.get('mockFunc') === mockFunc` — 함수가 이름 키로 Map에 저장됨

---

#### `it('throws A2uiExpressionError when function is not found')`

**검증 동작**: 존재하지 않는 함수명으로 `catalog.invoker`를 호출하면 `A2uiExpressionError`가 던져지는지 확인한다.

**픽스처/모킹**:
- `catalog`: 함수 없이 `new Catalog('test-cat', [])`
- `ctx`: `{} as any` — DataContext 더미

**검증 항목**:
- 던져진 에러가 `A2uiExpressionError`의 인스턴스임
- `err.message`에 `'Function not found'` 문자열이 포함됨
- `err.expression === 'nonExistent'` — 에러에 함수명이 첨부됨

---

#### `it('throws A2uiExpressionError when zod validation fails')`

**검증 동작**: 필수 필드 `requiredField`를 요구하는 스키마를 가진 함수에 빈 인수(`{}`)를 전달하면 Zod 검증 실패로 `A2uiExpressionError`가 던져지는지 확인한다.

**픽스처/모킹**:
- `mockFunc`: `createFunctionImplementation({ name: 'test', returnType: 'string', schema: z.object({ requiredField: z.string() }) }, () => 'result')`
- `catalog`: `new Catalog('test-cat', [], [mockFunc])`
- `ctx`: `{} as any`

**검증 항목**:
- 던져진 에러가 `A2uiExpressionError`의 인스턴스임
- `err.message`에 `'Validation failed'` 문자열이 포함됨
- `err.expression === 'test'` — 에러에 함수명이 첨부됨
- `Array.isArray(err.details)` — Zod 이슈 배열이 `details`에 포함됨

---

### `describe('InferredComponentApiSchemaType')`

#### `it('infers the correct schema type from a ComponentApi')`

**검증 동작**: `InferredComponentApiSchemaType<typeof mockApi>`가 `z.infer<typeof mockSchema>`와 동일한 타입을 추론하는지, 그리고 `any`로 퇴화하지 않는지를 타입 수준에서 검증한다.

**픽스처/모킹**:
- `mockSchema`: `z.object({ field1: z.string(), field2: z.number().optional() })`
- `mockApi`: `{ name: 'MockComp', schema: mockSchema } satisfies ComponentApi` — `satisfies` 키워드로 정확한 타입 좁힘

**검증 항목**:
1. `IsAny<InferredType>` 타입 검사를 위한 `inferredIsAny: IsAny<InferredType> = false` 대입이 성공 — `InferredType`이 `any`가 아님을 증명
2. `TypesAreEquivalent<ExpectedType, InferredType>` 검사를 위한 `typesMatchExact: TypesAreEquivalent<...> = true` 대입이 성공 — 두 타입이 상호 extends 관계임을 증명
3. `assert.strictEqual(inferredIsAny, false)`, `assert.strictEqual(typesMatchExact, true)` — 런타임에서 린터 경고 방지용 검증
