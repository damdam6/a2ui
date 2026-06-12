# renderers/lit/src/0.8/ui/utils/utils.ts

## 개요

이 파일은 A2UI v0.8 컴포넌트가 공통으로 사용하는 데이터 추출 유틸리티 함수를 제공한다. `Primitives.StringValue` 또는 `Primitives.NumberValue` 타입의 값에서 실제 문자열이나 숫자를 추출하는 두 함수를 export한다. 값이 리터럴 형태인지, 데이터 경로(path) 참조 형태인지에 따라 분기 처리하며, 경로 참조의 경우 `A2uiMessageProcessor`를 통해 런타임 모델 데이터를 동적으로 조회한다. 모든 오류 상황에는 안전한 기본값을 반환하여 렌더링 오류를 방지한다.

## 의존성

### 외부 패키지

- `@a2ui/web_core/data/model-processor` — `A2uiMessageProcessor`
- `@a2ui/web_core/types/primitives` — `Primitives` 네임스페이스 (`StringValue`, `NumberValue`)
- `@a2ui/web_core/types/types` — `Types` 네임스페이스 (`AnyComponentNode`)

### 저장소 내부 모듈

없음

## Exports

| 이름 | 종류 |
|---|---|
| `extractStringValue` | 함수 |
| `extractNumberValue` | 함수 |

## 상세 명세

### 함수: `extractStringValue`

- **시그니처:**
  `extractStringValue(val: Primitives.StringValue | null, component: Types.AnyComponentNode | null, processor: A2uiMessageProcessor | null, surfaceId: string | null): string`
- **매개변수:**
  - `val`: 추출 대상 값. `null`이거나 `StringValue` 객체(`literalString`, `literal`, `path` 중 하나의 필드를 가짐)
  - `component`: 현재 컴포넌트 노드. 경로 참조 시 필요
  - `processor`: 런타임 모델 데이터 접근기. 경로 참조 시 필요
  - `surfaceId`: 서피스 식별자. `null`이면 `A2uiMessageProcessor.DEFAULT_SURFACE_ID`를 대신 사용
- **반환 타입:** `string`
- **동작 로직:**
  1. `val`이 `null`이거나 객체가 아니면 빈 문자열 `''`을 반환한다.
  2. `val`에 `literalString` 키가 있으면 `val.literalString ?? ''`을 반환한다.
  3. `val`에 `literal` 키가 있고 값이 `undefined`가 아니면 `val.literal ?? ''`을 반환한다.
  4. `val`에 `path` 키가 있고 값이 truthy이면:
     - `processor` 또는 `component`가 없으면 `'(no model)'`을 반환한다.
     - `processor.getData(component, val.path, surfaceId ?? DEFAULT_SURFACE_ID)`로 런타임 데이터를 조회한다.
     - 결과가 `null`이거나 `string`이 아니면 `''`을 반환한다.
     - 유효한 문자열이면 그 값을 반환한다.
  5. 어떤 분기에도 해당하지 않으면 `''`을 반환한다.

### 함수: `extractNumberValue`

- **시그니처:**
  `extractNumberValue(val: Primitives.NumberValue | null, component: Types.AnyComponentNode | null, processor: A2uiMessageProcessor | null, surfaceId: string | null): number`
- **매개변수:** `extractStringValue`와 동일 구조이나 `val`이 `Primitives.NumberValue | null` 타입
- **반환 타입:** `number`
- **동작 로직:**
  1. `val`이 `null`이거나 객체가 아니면 `0`을 반환한다.
  2. `val`에 `literalNumber` 키가 있으면 `val.literalNumber ?? 0`을 반환한다.
  3. `val`에 `literal` 키가 있고 `undefined`가 아니면 `val.literal ?? 0`을 반환한다.
  4. `val`에 `path` 키가 있고 값이 truthy이면:
     - `processor` 또는 `component`가 없으면 `-1`을 반환한다.
     - `processor.getData(...)`로 런타임 데이터를 조회한다.
     - 조회 결과가 `string`이면 `Number.parseInt(값, 10)`으로 변환하고, `NaN`이면 `null`로 처리한다.
     - 최종값이 `null`이거나 `number`가 아니면 `-1`을 반환한다.
     - 유효한 숫자이면 그 값을 반환한다.
  5. 어떤 분기에도 해당하지 않으면 `0`을 반환한다.
- **경계 케이스:** 경로 참조 시 processor/component 미제공 → `-1`, 유효하지 않은 문자열 또는 null 데이터 → `-1` 반환. 호출 측에서 `-1`을 "데이터 없음" 신호로 구별할 수 있다.

## 동작 흐름

두 함수 모두 동일한 3단계 분기 패턴을 따른다: (1) 타입 전용 리터럴 키로 직접 반환 → (2) 공통 `literal` 키로 직접 반환 → (3) `path` 키로 런타임 모델에서 동적 조회. 모든 케이스에서 null-safe하게 폴백 값을 반환하므로 컴포넌트는 렌더링 오류 없이 안전하게 사용할 수 있다.
