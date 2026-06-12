# renderers/web_core/src/v0_9/reactivity/signals.test.ts

## 개요

`FrameworkSignal` 인터페이스가 실제 프레임워크 시그널 구현체(Angular, Preact)와 올바르게 연동되는지를 검증하는 통합 테스트 파일이다. Angular와 Preact 두 가지 시그널 패러다임(`()` 호출 방식 vs `.value` 속성 방식)을 대표 사례로 삼아, 인터페이스의 모든 메서드가 양 프레임워크에서 동일한 계약(contract)을 만족하는지 확인한다.

## 의존성

### 외부 패키지
- `node:assert` — 검증 유틸리티
- `node:test` — `describe`, `it`
- `@preact/signals-core` — `Signal`, `computed`, `effect` (Preact 시그널 구현체)
- `@angular/core` — `signal`, `computed`, `Signal`, `WritableSignal`, `isSignal`, `effect` (Angular 시그널 구현체)

### 저장소 내부 모듈
- [`./signals`](./signals.ts.md) — `FrameworkSignal` 인터페이스

## 모듈 선언 병합

테스트 파일 내에서 `declare module './signals'`로 두 프레임워크의 시그널 타입을 등록한다:
- `SignalKinds<T>` — `angular: ASignal<T>`, `preact: PSignal<T>`
- `WritableSignalKinds<T>` — `angular: AWritableSignal<T>`, `preact: PSignal<T>`

## 픽스처

### `AngularSignal: FrameworkSignal<'angular'>`

Angular 시그널을 `FrameworkSignal` 인터페이스로 감싸는 어댑터 객체:
- `computed`: `aComputed(fn)` 호출
- `isSignal`: Angular `isSignal(val)` 호출
- `wrap`: `aSignal(val)` 호출
- `unwrap`: `val()` 함수 호출로 값 추출
- `set`: `signal.set(value)` 호출
- `effect`: `aEffect()`를 사용하되, Angular의 cleanup 등록 방식(`cleanupRegisterFn(cleanupCallback)`)에 맞게 래핑하고, 반환값은 `() => e.destroy()`

### `PreactSignal: FrameworkSignal<'preact'>`

Preact 시그널을 `FrameworkSignal` 인터페이스로 감싸는 어댑터 객체:
- `computed`: `pComputed(fn)` 호출
- `isSignal`: `val instanceof PSignal` 검사
- `wrap`: `new PSignal(val)` 생성
- `unwrap`: `val.value` 속성으로 값 추출
- `set`: `signal.value = value` 직접 할당
- `effect`: `pEffect(fn)` 직접 호출 (cleanupCallback 무시)

## 테스트 케이스

### `describe('Angular variation')`

#### `round trip wraps and unwraps successfully`
- 픽스처: `AngularSignal`
- 검증: `'hello'`를 `wrap`한 뒤 `unwrap`하면 원래 값이 반환됨을 확인한다.

#### `handles updates well`
- 픽스처: `AngularSignal`
- 검증: `wrap('first')`로 시그널을 생성하고, `computed(() => \`prefix ${signal()}\``)`를 파생한 뒤:
  - 초기 상태: `signal()`, `unwrap(signal)`, `computedVal()`, `unwrap(computedVal)` 모두 예상값과 일치함을 확인한다.
  - `set(signal, 'second')` 후: 모든 값이 `'second'` / `'prefix second'`로 갱신됨을 확인한다.

#### `describe('.isSignal()')`

##### `validates a signal`
- 검증: `wrap(val)`의 결과에 `isSignal()`을 적용하면 `true`가 반환됨을 확인한다.

##### `rejects a non-signal`
- 검증: 일반 문자열 `'hello'`에 `isSignal()`을 적용하면 `false`가 반환됨을 확인한다.

---

### `describe('Preact variation')`

#### `round trip wraps and unwraps successfully`
- 픽스처: `PreactSignal`
- 검증: `'hello'`를 `wrap`한 뒤 `unwrap`하면 원래 값이 반환됨을 확인한다.

#### `handles updates well`
- 픽스처: `PreactSignal`
- 검증: `wrap('first')`로 시그널 생성 후 `computed(() => \`prefix ${signal.value}\``)`를 파생한 뒤:
  - 초기 상태: `.value` 속성과 `unwrap()`이 모두 예상값과 일치함을 확인한다.
  - `set(signal, 'second')` 후: 모든 값이 `'second'` / `'prefix second'`로 갱신됨을 확인한다.

#### `describe('.isSignal()')`

##### `validates a signal`
- 검증: `wrap(val)`의 결과에 `isSignal()`을 적용하면 `true`가 반환됨을 확인한다.

##### `rejects a non-signal`
- 검증: 일반 문자열 `'hello'`에 `isSignal()`을 적용하면 `false`가 반환됨을 확인한다.

## 동작 흐름

Angular는 `()` 호출로 값을 읽고 `.set()`으로 값을 쓰는 방식이며, Preact는 `.value` 속성으로 읽고 쓰는 방식이다. 두 어댑터 객체는 이 차이를 `FrameworkSignal` 인터페이스의 `unwrap`/`set` 메서드 내부에서 추상화하여, 소비 측 코드가 프레임워크를 의식하지 않고 동일한 API를 사용할 수 있음을 증명한다.
