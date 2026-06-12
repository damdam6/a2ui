# renderers/angular/src/v0_9/core/types.ts

## 개요

A2UI Angular 렌더러 v0.9에서 컴포넌트 속성 바인딩에 사용되는 핵심 타입 정의 파일이다. `BoundProperty` 인터페이스가 A2UI 데이터 모델과 Angular Signal 사이의 계약을 정의하며, 컴포넌트 API 스키마를 Angular 컴포넌트 props 타입으로 변환하는 유틸리티 타입들도 포함한다. 이 파일의 타입들은 카탈로그 내 모든 UI 컴포넌트의 props 타입을 결정하는 기반이 된다.

## 의존성

### 외부 패키지
- `@angular/core` — `Signal`
- `zod` — `z`
- `@a2ui/web_core/v0_9` — `ComponentApi`, `DataBindingSchema`, `FunctionCallSchema`

### 저장소 내부 모듈
- [`./component-binder.service`](./component-binder.service.ts.md) — `Child`

## Exports

- `ComponentTemplate` (인터페이스)
- `BoundProperty<T>` (인터페이스)
- `ExtendedProps<ComponentProps>` (타입)
- `ComponentApiToProps<Api>` (타입)

## 상세 명세

### `interface ComponentTemplate`

자식 컴포넌트 컬렉션을 렌더링하는 데 사용되는 템플릿 데이터 구조체.

| 필드 | 타입 | 설명 |
|------|------|------|
| `id` | `string \| undefined` | 템플릿 식별자 |
| `path` | `string \| undefined` | 데이터 경로 |

---

### `interface BoundProperty<T = unknown>`

A2UI 데이터 컨텍스트와 Angular Signal을 연결하는 핵심 바인딩 인터페이스. `ComponentBinder`가 Angular 컴포넌트에 속성을 노출할 때 이 타입을 사용한다.

| 필드 | 타입 | 설명 |
|------|------|------|
| `readonly value` | `Signal<T>` | 현재 해석된 값을 담은 리액티브 Angular Signal. A2UI 데이터 모델이 변경될 때 자동으로 업데이트된다. |
| `readonly raw` | `unknown` | `ComponentModel`의 원시 값. 리터럴이거나 데이터 바인딩 경로 객체일 수 있다. |
| `readonly template` | `ComponentTemplate \| undefined` | 자식 컴포넌트를 템플릿으로 정의하는 "children" props에만 존재한다. |
| `readonly onUpdate` | `(newValue: T) => void` | 값 변경 콜백. 데이터 경로에 바인딩된 경우 해당 경로를 업데이트하고, 리터럴 값인 경우 일반적으로 no-op이다. |

---

### 내부 타입 (비공개 헬퍼)

#### `type DataBindingType`

`z.infer<typeof DataBindingSchema>` — Zod 스키마에서 추론한 데이터 바인딩 타입.

#### `type FunctionCallType`

`z.infer<typeof FunctionCallSchema>` — Zod 스키마에서 추론한 함수 호출 바인딩 타입.

#### `type DynamicSchemaValueToRaw<Input>`

`Exclude<Input, DataBindingType | FunctionCallType>` — 스키마 값에서 바인딩 타입(`DataBindingType`, `FunctionCallType`)을 제외하여 리터럴 값 타입만 남긴다.

#### `type InferredInterfaceToProps<InferredSchema>`

Zod 스키마에서 추론된 인터페이스를 `BoundProperty` 래핑된 props 타입으로 변환하는 매핑 타입. 각 키 `K`에 대해:
- `K`가 `'children'`이면 → `BoundProperty<Child[]>`
- `K`가 `'child'`, `'trigger'`, `'content'`이면 → `BoundProperty<Child>`
- 그 외 → `BoundProperty<DynamicSchemaValueToRaw<InferredSchema[K]>>` (바인딩 타입을 제거한 리터럴 타입)

#### `interface CheckProps` (비공개)

| 필드 | 타입 |
|------|------|
| `isValid` | `boolean` |
| `validationErrors` | `string[]` |

---

### `type ExtendedProps<ComponentProps extends { [key: string]: unknown }>`

`ComponentProps`에 `'checks'` 키가 존재하면 `ComponentProps & CheckProps`를 반환하고, 그렇지 않으면 `ComponentProps`를 그대로 반환한다. `ComponentBinder`가 유효성 검사 결과를 props에 추가할 때 타입에 반영하기 위한 유틸리티 타입이다.

---

### `type ComponentApiToProps<Api extends ComponentApi>`

컴포넌트 API 타입(`ComponentApi`)을 Angular 컴포넌트 props 타입으로 변환하는 최상위 유틸리티 타입.

변환 과정:
1. `Api['schema']`에서 Zod로 타입을 추론: `z.infer<Api['schema']>`
2. `ExtendedProps`를 적용하여 `checks` 필드가 있는 경우 `CheckProps`를 추가.
3. `InferredInterfaceToProps`를 적용하여 모든 필드를 `BoundProperty`로 감싼다.

**사용 예시** (문서 내 인라인 주석에서):
```
// TextComponentApi의 schema가 z.object({ text: z.string() })이면
// ComponentApiToProps<typeof TextComponentApi> === { text: BoundProperty<string>; }
```

## 동작 흐름

이 파일은 순수 타입 정의 파일로 런타임 로직이 없다. 컴파일 타임에 카탈로그 컴포넌트들이 자신의 props 타입을 `ComponentApiToProps`를 통해 자동으로 도출할 수 있게 하며, `BoundProperty` 인터페이스를 통해 A2UI 데이터 바인딩 시스템과 Angular Signal 시스템 간의 계약을 명확히 정의한다.
