# specification/v0_8/eval/src/surface_update_schema_matcher.ts

## 개요

이 파일은 `surfaceUpdate` 메시지의 `components` 배열에서 특정 컴포넌트 타입이 존재하는지, 그리고 선택적으로 해당 컴포넌트의 특정 프로퍼티가 기대값을 포함하는지 검증하는 `SurfaceUpdateSchemaMatcher` 클래스를 구현한다. bound value 객체(`literalString`, `literalNumber`, `literalBoolean`), 배열 옵션, Button의 child Text 컴포넌트를 통한 label 검사 등 복잡한 값 구조를 지원한다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈
- [`./schema_matcher`](./schema_matcher.ts.md) — `SchemaMatcher`, `ValidationResult`

## Exports

| 이름 | 종류 | 설명 |
|------|------|------|
| `SurfaceUpdateSchemaMatcher` | class | `surfaceUpdate` 메시지 컴포넌트 검증 매처 |

## 상세 명세

### `SurfaceUpdateSchemaMatcher` (class, extends `SchemaMatcher`)

#### 필드 및 생성자

```
constructor(
  public componentType: string,
  public propertyName?: string,
  public propertyValue?: any,
  public caseInsensitive: boolean = false,
)
```

- `componentType: string` — 찾을 컴포넌트 타입 이름. 예: `'Button'`, `'TextField'`, `'Heading'`.
- `propertyName?: string` — 선택. 컴포넌트 내에서 검사할 프로퍼티 이름.
- `propertyValue?: any` — 선택. `propertyName`과 함께 제공 시 해당 프로퍼티 값을 비교한다.
- `caseInsensitive: boolean` — 기본값 `false`. `true`이면 문자열 비교 시 대소문자를 무시한다.

#### `validate(schema: any): ValidationResult`

검증은 단계적으로 진행된다.

1. `schema.surfaceUpdate`가 없으면 즉시 실패 반환.
2. `schema.surfaceUpdate.components`가 배열이 아니면 실패 반환.
3. `components` 배열을 순회하며 각 컴포넌트 항목(`c`)의 `c.component`에 두 개 이상의 키가 있는 경우 오류 반환. (컴포넌트 항목 하나에는 단일 타입만 허용)
4. `c.component[componentType]`이 존재하는 항목들을 `matchingComponents`로 필터링한다.
5. `matchingComponents`가 비어 있으면 `"Failed to find component of type '${componentType}'."` 오류 반환.
6. `propertyName`이 없으면 `{success: true}` 반환(타입 존재 확인만 요구).
7. `matchingComponents`를 순회하며 각 컴포넌트에 대해:
   - `properties[propertyName]`이 `undefined`가 아니면 `propertyValue`가 없거나(`undefined`) `valueMatches()` 판정이 참이면 즉시 `{success: true}` 반환.
   - `componentType === 'Button'`이고 `propertyName === 'label'`이며 `properties.child`가 있으면, `child` ID로 `getComponentById()`를 통해 참조 컴포넌트를 찾아 그것이 `Text` 타입이고 `text` 값이 `valueMatches()` 판정을 통과하면 즉시 성공 반환.
8. 모든 컴포넌트를 확인해도 일치하지 않으면 `propertyValue` 유무에 따라 오류 메시지를 달리하여 실패 반환.

#### `private getComponentById(components: any[], id: string): any | undefined`

`components` 배열에서 `c.id === id`인 첫 번째 항목을 찾아 반환한다. 없으면 `undefined`.

#### `private valueMatches(actualValue: any, expectedValue: any): boolean`

`actualValue`가 `null` 또는 `undefined`이면 `false` 반환.

문자열 비교 보조 함수 `compareStrings`를 내부적으로 정의한다. `caseInsensitive`가 `true`이면 양쪽을 소문자로 변환 후 비교, 아니면 직접 비교(`===`)한다.

값 형태에 따라 다음 순서로 처리한다.

- **객체(비배열)인 경우**: `literalString` 키가 있으면 `expectedValue`가 문자열인지 확인 후 `compareStrings`로 비교. `literalNumber` 키가 있으면 `===` 비교. `literalBoolean` 키가 있으면 `===` 비교.
- **배열인 경우**: 각 item을 순회한다.
  - item이 객체이면: `literalString` / `literalNumber` / `literalBoolean` 직접 값 비교 시도; `item.label.literalString`과 `expectedValue` 비교(`compareStrings`); `item.value === expectedValue` 비교.
  - item이 문자열이면 `compareStrings`로 비교.
  - 그 외: `===` 직접 비교.
  - 어느 item에서든 일치하면 즉시 `true` 반환.
- **폴백**: `JSON.stringify(actualValue) === JSON.stringify(expectedValue)`

## 동작 흐름

이 매처는 LLM이 생성한 `surfaceUpdate` JSON이 요구하는 컴포넌트를 포함하는지 자동 검증하는 핵심 컴포넌트다. 컴포넌트 타입만 검사하거나(3인자), 특정 프로퍼티 존재 여부만 검사하거나(4인자), 프로퍼티 값까지 검사(5인자)하는 세 가지 사용 수준을 지원한다. Button의 label이 직접 속성이 아닌 child Text 컴포넌트에 담긴 경우를 별도로 처리하여 A2UI 스키마의 두 가지 Button 표현 방식을 모두 허용한다.
