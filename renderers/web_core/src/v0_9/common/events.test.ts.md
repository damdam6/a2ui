# renderers/web_core/src/v0_9/common/events.test.ts

## 개요

`common/events.ts`의 `EventEmitter` 클래스 동작을 검증하는 단위 테스트 파일이다. Node.js 내장 테스트 러너(`node:test`)와 `node:assert`를 사용한다. 구독/해제 동작과 리스너 예외 처리 동작 두 가지를 테스트한다.

## 의존성

### 외부 패키지
없음 (Node.js 내장 모듈 `node:assert`, `node:test` 사용)

### 저장소 내부 모듈

- [`./events.js`](./events.ts.md) — `EventEmitter`

## 테스트 케이스 명세

### `describe('Events')`

#### `it('handles subscriptions and unsubscriptions correctly')`

**검증 동작**: `EventEmitter`의 구독 추가 및 해제가 올바르게 동작하는지 확인한다.

**픽스처**:
- `emitter`: `new EventEmitter<string>()`
- `callCount`: 숫자 카운터 (초기값 `0`)
- `lastValue`: 문자열 변수 (초기값 `''`)
- `listener`: `callCount`를 증가시키고 `lastValue`를 갱신하는 함수

**검증 흐름**:
1. `emitter.subscribe(listener)` 호출 → `sub` 반환
2. `await emitter.emit('hello')` — 리스너 호출됨
3. `callCount === 1`, `lastValue === 'hello'` 검증
4. `sub.unsubscribe()` 호출 — 구독 해제
5. `await emitter.emit('world')` — 리스너가 호출되지 않아야 함
6. `callCount === 1` (증가 없음), `lastValue === 'hello'` (변경 없음) 검증

---

#### `it('handles errors thrown by listeners')`

**검증 동작**: 리스너가 예외를 던져도 `EventEmitter`가 충돌하지 않고 `console.error`로 오류를 로깅하는지 확인한다.

**픽스처/모킹**:
- `emitter`: `new EventEmitter<string>()`
- `originalConsoleError`: 원본 `console.error` 저장
- `errorLogged`: boolean (초기값 `false`)
- `console.error` 임시 교체: 메시지가 `'EventEmitter error:'`이고 에러 메시지가 `'Test Error'`인 경우 `errorLogged = true`로 설정

**검증 흐름**:
1. `emitter.subscribe(() => { throw new Error('Test Error'); })` — 예외를 던지는 리스너 등록
2. `await emitter.emit('trigger')` — emit 자체는 성공해야 함
3. `errorLogged === true` 검증 — `console.error('EventEmitter error:', e)` 가 호출됐음을 확인
4. `finally` 블록에서 `console.error` 원복

