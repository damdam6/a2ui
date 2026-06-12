# renderers/web_core/src/v0_9/schema/client-to-server.test.ts

## 개요

`client-to-server.ts`에 정의된 Zod 스키마(`A2uiClientMessageSchema`, `A2uiClientDataModelSchema`)를 `node:test`와 `node:assert`를 사용해 검증하는 테스트 파일이다. 유효한 액션 메시지, 유효성 검사 오류 메시지, 일반 오류 메시지, 데이터 모델 메시지의 파싱 성공을 확인하고, 잘못된 버전 문자열이 올바르게 거부되는지 확인한다.

## 의존성

### 외부 패키지
- `node:test` (`describe`, `it`)
- `node:assert`

### 저장소 내부 모듈
- [`./client-to-server.js`](./client-to-server.ts.md) — `A2uiClientMessageSchema`, `A2uiClientDataModelSchema`

## Exports

없음. 테스트 파일로 실행 전용이다.

## 테스트 케이스 명세

모든 테스트는 `describe('Client-to-Server Schema Verification', ...)` 블록 내에 있다.

---

### `it('validates a valid action message')`

**검증하는 동작**: `version: 'v0.9'`와 `action` 필드를 가진 객체가 `A2uiClientMessageSchema`를 통과하는지 확인한다.

**픽스처**: `action` 객체에 `name: 'submit'`, `surfaceId: 's1'`, `sourceComponentId: 'c1'`, `timestamp: new Date().toISOString()`, `context: {foo: 'bar'}`를 포함한다.

**검증 방법**: `safeParse` 결과의 `result.success`가 `true`인지 `assert.ok`로 확인한다. 실패 시 `result.error.message`를 출력한다.

---

### `it('validates a valid error message (validation failed)')`

**검증하는 동작**: `code: 'VALIDATION_FAILED'`와 `path`, `message`, `surfaceId`를 가진 오류 메시지가 `A2uiClientMessageSchema`를 통과하는지 확인한다.

**픽스처**: `version: 'v0.9'`, `error: { code: 'VALIDATION_FAILED', surfaceId: 's1', path: '/components/0/text', message: 'Too short' }`.

**검증 방법**: `safeParse` 결과의 `result.success`가 `true`인지 `assert.ok`로 확인한다.

---

### `it('validates a valid error message (generic)')`

**검증하는 동작**: `code`가 `'VALIDATION_FAILED'`가 아닌 일반 오류 메시지가 `A2uiClientMessageSchema`를 통과하는지 확인한다.

**픽스처**: `version: 'v0.9'`, `error: { code: 'INTERNAL_ERROR', message: 'Something went wrong', surfaceId: 's1' }`.

**검증 방법**: `safeParse` 결과의 `result.success`가 `true`인지 `assert.ok`로 확인한다.

---

### `it('validates a valid data model message')`

**검증하는 동작**: `version: 'v0.9'`와 `surfaces` 맵을 가진 데이터 모델 메시지가 `A2uiClientDataModelSchema`를 통과하는지 확인한다.

**픽스처**: `surfaces: { s1: {user: 'Alice'}, s2: {cart: []} }`.

**검증 방법**: `safeParse` 결과의 `result.success`가 `true`인지 `assert.ok`로 확인한다.

---

### `it('fails on invalid version')`

**검증하는 동작**: `version: 'v0.8'`처럼 지원하지 않는 버전이 거부되는지 확인한다.

**픽스처**: `action` 객체는 유효하지만 `version: 'v0.8'`로 설정한 객체.

**검증 방법**: `safeParse` 결과의 `result.success`가 `false`인지 `assert.strictEqual`로 확인한다.

## 동작 흐름

각 테스트는 `A2uiClientMessageSchema.safeParse(...)` 또는 `A2uiClientDataModelSchema.safeParse(...)`를 직접 호출한다. 모킹이나 픽스처 설정 함수 없이 인라인 객체 리터럴을 사용한다.
