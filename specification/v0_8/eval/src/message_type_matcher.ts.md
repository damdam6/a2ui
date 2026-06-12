# specification/v0_8/eval/src/message_type_matcher.ts

## 개요

이 파일은 응답 JSON의 최상위 키가 단 하나이고 그 키가 지정한 메시지 타입과 일치하는지 검증하는 `MessageTypeMatcher` 클래스를 구현한다. A2UI 프로토콜 메시지는 최상위 레벨에 단일 타입 키(예: `surfaceUpdate`, `deleteSurface`, `dataModelUpdate`, `beginRendering`)를 가져야 하는 규칙을 강제한다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈
- [`./schema_matcher`](./schema_matcher.ts.md) — `SchemaMatcher`, `ValidationResult`

## Exports

| 이름 | 종류 | 설명 |
|------|------|------|
| `MessageTypeMatcher` | class | 최상위 메시지 타입 키 검증 매처 |

## 상세 명세

### `MessageTypeMatcher` (class, extends `SchemaMatcher`)

#### 필드 및 생성자

```
constructor(private messageType: string)
```

- `messageType: string` — 기대하는 최상위 메시지 타입 키 이름. private 필드로 저장된다.

#### `validate(response: object): ValidationResult`

1. `response`가 falsy이거나 `typeof response !== 'object'`이면 `{success: false, error: 'Response is not a valid object.'}` 반환.
2. `Object.keys(response)`로 최상위 키 목록을 추출한다.
3. 키가 정확히 하나이고 그 키가 `messageType`과 일치하면 `{success: true}` 반환.
4. 그렇지 않으면 실제 키 목록을 쉼표로 연결하여 오류 메시지를 구성하고 실패 결과를 반환한다. 오류 형식: `"Expected top-level message type to be '${messageType}', but found '${keys.join(', ')}'"`

#### `get description(): string`

`"Expected top-level message type to be '${messageType}'"` 문자열을 반환하는 getter. 진단 목적으로 사용될 수 있다.

## 동작 흐름

이 매처는 A2UI 서버→클라이언트 메시지가 정확히 하나의 최상위 타입 키를 가지는 규칙을 강제한다. 키가 없거나 두 개 이상인 경우 모두 실패로 처리된다.
