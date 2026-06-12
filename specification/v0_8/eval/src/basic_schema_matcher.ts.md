# specification/v0_8/eval/src/basic_schema_matcher.ts

## 개요

이 파일은 임의의 JSON 객체에서 점 표기법(dot-notation) 경로로 특정 프로퍼티가 존재하는지, 그리고 선택적으로 그 값이 기대값과 일치하는지 검증하는 `BasicSchemaMatcher` 클래스를 구현한다. `SchemaMatcher` 추상 클래스를 상속하며, 주로 최상위 메시지 타입의 단순 프로퍼티를 검사하는 데 사용된다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈
- [`./schema_matcher`](./schema_matcher.ts.md) — `SchemaMatcher`, `ValidationResult`

## Exports

| 이름 | 종류 | 설명 |
|------|------|------|
| `BasicSchemaMatcher` | class | 점 표기법 경로 기반 프로퍼티 존재 및 값 검증 매처 |

## 상세 명세

### `BasicSchemaMatcher` (class, extends `SchemaMatcher`)

#### 필드 및 생성자

```
constructor(public propertyPath: string, public propertyValue?: any)
```

- `propertyPath: string` — 검사할 프로퍼티 경로. 점(`.`)으로 구분된 계층 경로를 허용한다. 예: `'deleteSurface.surfaceId'`.
- `propertyValue?: any` — 선택 인자. 제공되면 실제 값과 비교 검증을 수행한다. 생략하면 프로퍼티 존재 여부만 검사한다.

#### `validate(schema: any): ValidationResult`

1. `schema`가 falsy이면 `{success: false, error: 'Schema is undefined.'}` 반환.
2. `propertyPath`를 `.`으로 분리하여 `pathParts` 배열을 만든다.
3. `actualValue`를 `schema`로 초기화한 뒤, `pathParts`를 순서대로 순회하며 각 파트를 키로 하여 객체를 한 단계씩 내려간다. 중간 단계에서 값이 `null`이거나 객체가 아닐 경우 `actualValue`를 `undefined`로 설정하고 루프를 즉시 중단한다.
4. `actualValue`가 `undefined`이면 `{success: false, error: "Failed to find property '${propertyPath}'."}` 반환.
5. `propertyValue`가 제공된 경우(`!== undefined`), `JSON.stringify(actualValue)`와 `JSON.stringify(propertyValue)`를 비교한다. 불일치하면 실제 값과 기대 값을 포함한 오류 메시지와 함께 실패 결과를 반환한다.
6. 모든 검사를 통과하면 `{success: true}` 반환.

## 동작 흐름

인스턴스 생성 시 경로와 기대 값을 저장한다. `validate()`가 호출되면 경로를 탐색하여 값을 추출하고, 기대 값이 제공된 경우 JSON 직렬화 비교를 통해 동등성을 판단한다. 배열 인덱스도 경로에 숫자 문자열(예: `'0'`)로 포함할 수 있어 `dataModelUpdate.contents.0.key` 같은 경로도 지원한다.
