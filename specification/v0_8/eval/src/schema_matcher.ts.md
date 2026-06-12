# specification/v0_8/eval/src/schema_matcher.ts

## 개요

이 파일은 평가(eval) 시스템 전체에서 사용하는 검증 결과 타입과 추상 매처 기반 클래스를 정의한다. `ValidationResult` 인터페이스와 `SchemaMatcher` 추상 클래스를 export하여, 구체적인 매처 구현체(`BasicSchemaMatcher`, `MessageTypeMatcher`, `SurfaceUpdateSchemaMatcher`)가 공통 계약을 따르도록 강제한다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈
없음

## Exports

| 이름 | 종류 | 설명 |
|------|------|------|
| `ValidationResult` | interface | 검증 성공 여부와 선택적 오류 메시지를 담는 결과 타입 |
| `SchemaMatcher` | abstract class | `validate` 추상 메서드를 강제하는 기반 클래스 |

## 상세 명세

### `ValidationResult` (interface)

두 필드를 가지는 검증 결과 객체다.

- `success: boolean` — 검증 성공 여부. 필수 필드.
- `error?: string` — 실패 시 사람이 읽을 수 있는 오류 설명. 선택 필드로, 성공 시에는 생략한다.

### `SchemaMatcher` (abstract class)

모든 구체적 매처가 상속해야 하는 추상 클래스다.

- **추상 메서드** `validate(schema: any): ValidationResult` — 주어진 `schema` 객체를 검사하여 `ValidationResult`를 반환한다. 구체적인 검증 로직은 서브클래스에서 구현한다.
- 생성자와 필드는 별도로 정의하지 않는다.

## 동작 흐름

이 파일 자체는 런타임 로직을 포함하지 않는다. 타입 및 계약을 선언하는 역할만 하며, 다른 매처 파일들이 이 파일을 import하여 `SchemaMatcher`를 extends하고 `ValidationResult`를 반환 타입으로 사용한다.
