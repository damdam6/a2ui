# renderers/web_core/src/v0_8/errors.ts

## 개요

A2UI 라이브러리 전용 에러 클래스 계층을 정의한다. `A2uiError`를 기반 클래스로 두고, 검증 실패(`A2uiValidationError`), 데이터 모델 변이 오류(`A2uiDataError`), 표현식 평가 오류(`A2uiExpressionError`), UI 트리 구조 오류(`A2uiStateError`) 네 가지 특화 클래스를 export한다. 각 클래스는 고정된 `code` 문자열을 통해 프로그램적으로 에러 종류를 구분할 수 있게 한다.

## 의존성

### 외부 패키지
- (없음)

### 저장소 내부 모듈
- (없음)

## Exports

| 이름 | 종류 |
|------|------|
| `A2uiError` | 클래스 (기반 클래스) |
| `A2uiValidationError` | 클래스 |
| `A2uiDataError` | 클래스 |
| `A2uiExpressionError` | 클래스 |
| `A2uiStateError` | 클래스 |

## 상세 명세

### 내부 인터페이스 `V8ErrorConstructor` (비공개)

`ErrorConstructor`를 확장해 V8 전용 정적 메서드 `captureStackTrace(targetObject: object, constructorOpt?: Function): void`를 선언한다. 표준 `Error` 타입에는 이 메서드가 없으므로, 타입 캐스팅을 통해 접근하기 위해 필요하다.

---

### 클래스 `A2uiError extends Error`

모든 A2UI 에러의 기반 클래스.

#### 필드

- `public readonly code: string` — 에러 종류를 식별하는 문자열 코드.

#### 생성자 `constructor(message: string, code: string = 'UNKNOWN_ERROR')`

1. `super(message)`로 `Error`를 초기화한다.
2. `this.name = this.constructor.name`으로 에러 이름을 실제 클래스명으로 설정한다(스택 트레이스 가독성).
3. `this.code = code`로 코드를 할당한다.
4. `(Error as V8ErrorConstructor).captureStackTrace`가 존재하면 `captureStackTrace(this, this.constructor)`를 호출해 V8 환경에서 생성자가 스택 프레임에 포함되지 않도록 한다.

---

### 클래스 `A2uiValidationError extends A2uiError`

JSON 검증 실패 또는 스키마 불일치 시 던진다.

#### 생성자 `constructor(message: string, details?: any)`

- `super(message, 'VALIDATION_ERROR')` 호출.
- `public readonly details?: any` — 추가 검증 세부 정보(옵션).

---

### 클래스 `A2uiDataError extends A2uiError`

데이터 모델 변이 중 유효하지 않은 경로나 타입 불일치 발생 시 던진다.

#### 생성자 `constructor(message: string, path?: string)`

- `super(message, 'DATA_ERROR')` 호출.
- `public readonly path?: string` — 문제가 발생한 데이터 경로(옵션).

---

### 클래스 `A2uiExpressionError extends A2uiError`

문자열 보간 또는 함수 평가 중 오류 발생 시 던진다.

#### 생성자 `constructor(message: string, expression?: string)`

- `super(message, 'EXPRESSION_ERROR')` 호출.
- `public readonly expression?: string` — 오류를 발생시킨 표현식 문자열(옵션).

---

### 클래스 `A2uiStateError extends A2uiError`

UI 트리의 구조적 문제(누락된 서페이스, 중복 컴포넌트, 순환 의존성 등) 발생 시 던진다.

#### 생성자 `constructor(message: string)`

- `super(message, 'STATE_ERROR')` 호출. 추가 필드 없음.

## 동작 흐름

파일은 클래스 계층을 선언만 한다. 런타임 제어 흐름 없이, `throw new A2uiXxxError(...)` 형태로 호출 측에서 사용한다. `code` 필드 덕분에 `catch` 블록에서 `instanceof` 대신 `error.code === 'VALIDATION_ERROR'`와 같은 문자열 비교도 가능하다.
