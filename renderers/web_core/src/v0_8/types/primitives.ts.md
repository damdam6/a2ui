# renderers/web_core/src/v0_8/types/primitives.ts

## 개요

A2UI 데이터 모델에서 사용하는 세 가지 원시(primitive) 값 래퍼 타입(`StringValue`, `NumberValue`, `BooleanValue`)을 Zod 스키마로부터 도출하여 내보내는 파일이다. 컴포넌트 속성이나 데이터 바인딩에서 단순 원시값을 구조화된 객체로 표현할 때 사용된다. 런타임 로직은 없으며 타입 선언만 포함한다.

## 의존성

- 외부 패키지: `zod` (`z` 타입 임포트)
- 저장소 내부 모듈:
  - `../schema/common-types.ts` — `StringValueSchema`, `NumberValueSchema`, `BooleanValueSchema` 타입 임포트

## Exports

| 이름 | 종류 | 대응 스키마 |
|------|------|------|
| `StringValue` | 인터페이스 | `StringValueSchema` |
| `NumberValue` | 인터페이스 | `NumberValueSchema` |
| `BooleanValue` | 인터페이스 | `BooleanValueSchema` |

## 상세 명세

세 인터페이스 모두 `export declare interface Xxx extends z.infer<typeof XxxSchema> {}` 형태로 선언된다. 구체적인 필드 구조는 `../schema/common-types.ts`의 Zod 스키마에 의해 결정된다.

- `StringValue`: 문자열 원시값을 담는 래퍼 타입
- `NumberValue`: 숫자 원시값을 담는 래퍼 타입
- `BooleanValue`: 불리언 원시값을 담는 래퍼 타입

## 동작 흐름

컴파일 타임 전용 파일이다. `types.ts`를 포함한 상위 모듈에서 이 세 가지 타입을 임포트하여 데이터 모델의 값 타입을 표현하는 데 사용한다.
