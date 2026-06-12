# renderers/web_core/src/v0_9/reactivity/signals.ts

## 개요

프레임워크 독립적인 시그널(signal) 추상화 계층을 정의한다. `SignalKinds`와 `WritableSignalKinds` 인터페이스는 모듈 선언 병합(declaration merging)을 통해 외부 라이브러리가 자신의 시그널 구현체를 타입 안전하게 등록할 수 있도록 설계되었다. `FrameworkSignal<K>` 인터페이스는 이 추상화의 통합 진입점으로, 래핑·언래핑·computed·effect·set·isSignal 기능을 정의한다.

## 의존성

외부 패키지 및 저장소 내부 모듈 없음. 순수 TypeScript 타입 정의 파일이다.

## Exports

| 이름 | 종류 |
|------|------|
| `SignalKinds<T>` | 인터페이스 (확장 포인트) |
| `WritableSignalKinds<T>` | 인터페이스 (확장 포인트) |
| `FrameworkSignal<K>` | 인터페이스 |

## 상세 명세

### 인터페이스 `SignalKinds<T>`

빈 인터페이스로, 타입 매개변수 `T`는 사용되지 않지만 하위 구현체가 시그널 타입에 제네릭을 전달하기 위해 필수적으로 선언해야 한다. 외부 모듈은 `declare module '...signals'` 선언 병합을 통해 키-값 쌍(예: `preact: Signal<T>`)을 추가하여 지원하는 시그널 종류를 등록한다.

### 인터페이스 `WritableSignalKinds<T>`

`SignalKinds<T>`와 동일한 구조로, 쓰기 가능한(writable) 시그널 타입을 별도로 분리하여 관리한다. 읽기 전용 computed 시그널과 쓰기 가능 원시 시그널을 구분하기 위해 존재한다.

### 인터페이스 `FrameworkSignal<K extends keyof SignalKinds<any>>`

특정 프레임워크 시그널 구현체에 대한 어댑터 계약(contract)이다. 타입 매개변수 `K`는 등록된 시그널 종류의 키(예: `'angular'`, `'preact'`)이다.

#### 메서드 목록

| 시그니처 | 설명 |
|---------|------|
| `computed<T>(fn: () => T): SignalKinds<T>[K]` | 파생(derived) 시그널을 생성한다. `fn`은 의존 시그널을 읽는 함수이며, 반환값은 해당 프레임워크의 computed 시그널 타입이다. |
| `effect(fn: () => void, cleanupCallback?: () => void): () => void` | 리액티브 이펙트를 실행한다. 의존성이 변경될 때마다 `fn`이 재실행된다. `cleanupCallback`은 이펙트 정리 시 호출된다. 반환값은 이펙트를 중지하는 함수이다. |
| `isSignal(val: unknown): val is SignalKinds<any>[K]` | 임의의 값이 이 프레임워크의 시그널 타입인지 타입 가드로 검사한다. |
| `wrap<T>(val: T): WritableSignalKinds<T>[K]` | 일반 값을 쓰기 가능 시그널로 감싼다. |
| `unwrap<T>(val: SignalKinds<T>[K]): T` | 시그널에서 현재 값을 꺼낸다. |
| `set<T>(signal: WritableSignalKinds<T>[K], value: T): void` | 쓰기 가능 시그널의 값을 갱신한다. |

## 동작 흐름

이 파일은 런타임 로직 없이 타입 계약만 정의한다. 구체적인 구현체(Angular, Preact 등)는 `declare module` 블록으로 `SignalKinds`/`WritableSignalKinds`에 자신의 타입을 등록하고, `FrameworkSignal<K>`를 구현한 객체를 제공한다. 이를 통해 web_core의 나머지 코드는 특정 프레임워크에 종속되지 않고 시그널을 다룰 수 있다.
