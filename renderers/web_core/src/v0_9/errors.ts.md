# renderers/web_core/src/v0_9/errors.ts

## 개요

A2UI 전용 에러 클래스 계층 구조를 정의하는 파일이다. 모든 A2UI 에러의 기반인 `A2uiError`를 최상위로, 검증 오류(`A2uiValidationError`), 데이터 오류(`A2uiDataError`), 표현식 오류(`A2uiExpressionError`), 상태 오류(`A2uiStateError`) 네 가지 특화 클래스를 제공한다. 각 클래스는 기계 판독 가능한 `code` 문자열을 포함하고, V8 환경에서 올바른 스택 트레이스를 생성한다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈
없음

## Exports

| 이름 | 종류 | 설명 |
|---|---|---|
| `A2uiError` | 클래스 | 모든 A2UI 에러의 기반 클래스 |
| `A2uiValidationError` | 클래스 | JSON 검증 실패 또는 스키마 불일치 시 던져지는 에러 |
| `A2uiDataError` | 클래스 | DataModel 변경 시(잘못된 경로, 타입 불일치) 던져지는 에러 |
| `A2uiExpressionError` | 클래스 | 문자열 보간 및 함수 평가 중 던져지는 에러 |
| `A2uiStateError` | 클래스 | UI 트리 구조 이상(서피스 누락, 컴포넌트 중복) 시 던져지는 에러 |

## 상세 명세

### 비공개 인터페이스: `V8ErrorConstructor extends ErrorConstructor`

`captureStackTrace(targetObject: object, constructorOpt?: Function): void` 메서드를 선언한다. V8 엔진에서만 존재하는 `Error.captureStackTrace`를 TypeScript에서 안전하게 접근하기 위한 내부 타입이다.

---

### `class A2uiError extends Error`

#### 필드

| 필드 | 타입 | 설명 |
|---|---|---|
| `code` | `public readonly string` | 에러 범주를 나타내는 기계 판독 가능한 문자열 |

#### `constructor(message: string, code: string = 'UNKNOWN_ERROR')`

1. `super(message)` 호출
2. `this.name = this.constructor.name` — 에러 이름을 클래스 이름으로 설정 (서브클래스 포함)
3. `this.code = code` — 코드 저장
4. V8 환경에서 `Error.captureStackTrace`가 존재하면 `this.constructor`를 두 번째 인수로 전달해 호출하여, 에러 생성자 내부 프레임이 스택 트레이스에서 제거된다.

---

### `class A2uiValidationError extends A2uiError`

JSON 검증 실패 또는 스키마 불일치 시 사용.

#### 필드

| 필드 | 타입 | 설명 |
|---|---|---|
| `details` | `public readonly any` (optional) | 검증 오류 세부 정보 |

#### `constructor(message: string, details?: any)`

- `super(message, 'VALIDATION_ERROR')` 호출
- `code`는 `'VALIDATION_ERROR'`로 고정됨

---

### `class A2uiDataError extends A2uiError`

DataModel 변경 중 잘못된 경로나 타입 불일치 시 사용.

#### 필드

| 필드 | 타입 | 설명 |
|---|---|---|
| `path` | `public readonly string` (optional) | 오류가 발생한 데이터 경로 |

#### `constructor(message: string, path?: string)`

- `super(message, 'DATA_ERROR')` 호출
- `code`는 `'DATA_ERROR'`로 고정됨

---

### `class A2uiExpressionError extends A2uiError`

문자열 보간 및 함수 평가 중 사용. `Catalog.invoker`에서 함수 미발견 또는 Zod 검증 실패 시 던져진다.

#### 필드

| 필드 | 타입 | 설명 |
|---|---|---|
| `expression` | `public readonly string` (optional) | 오류가 발생한 표현식 또는 함수명 |
| `details` | `public readonly any` (optional) | 추가 세부 정보 (예: Zod 이슈 배열) |

#### `constructor(message: string, expression?: string, details?: any)`

- `super(message, 'EXPRESSION_ERROR')` 호출
- `code`는 `'EXPRESSION_ERROR'`로 고정됨

---

### `class A2uiStateError extends A2uiError`

UI 트리 구조 이상(서피스 누락, 컴포넌트 중복 등) 시 사용.

#### `constructor(message: string)`

- `super(message, 'STATE_ERROR')` 호출
- `code`는 `'STATE_ERROR'`로 고정됨
- 추가 필드 없음

## 동작 흐름

이 파일은 순수한 클래스 정의만 포함한다. 런타임에 발생하는 오류를 의미별로 구분할 수 있도록 각 클래스가 고유한 `code` 값을 갖는다. 소비자는 `instanceof` 검사로 에러 유형을 구분하고 `code`, `expression`, `details`, `path` 등의 필드로 오류 맥락을 파악한다.
