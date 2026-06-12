# renderers/lit/src/0.8/data/signal-model-processor.ts

## 개요

`@a2ui/web_core`의 `A2uiMessageProcessor`를 `signal-utils` 패키지의 반응형(Signal) 컬렉션 생성자로 초기화하는 팩토리 함수 `create`를 제공하는 모듈이다. Lit의 반응형 렌더링 사이클과 통합될 수 있도록 일반 JS 내장 컬렉션(`Array`, `Map`, `Object`, `Set`) 대신 Signal 버전 컬렉션을 주입한다.

## 의존성

### 저장소 내부 모듈
없음 (모두 외부 패키지 의존)

### 외부 패키지
- `@a2ui/web_core/data/model-processor` — `A2uiMessageProcessor` 클래스
- `signal-utils/array` — `SignalArray`
- `signal-utils/map` — `SignalMap`
- `signal-utils/object` — `SignalObject`
- `signal-utils/set` — `SignalSet`

## Exports

- `create` (함수): `A2uiMessageProcessor` 인스턴스를 생성해 반환하는 팩토리 함수

## 상세 명세

### `create(): A2uiMessageProcessor`

**시그니처**: 매개변수 없음, 반환 타입 `A2uiMessageProcessor` (암묵적 추론)

**동작 로직**:
1. `new A2uiMessageProcessor({...})` 호출로 인스턴스를 생성한다.
2. 생성자 옵션 객체에 네 개의 컬렉션 생성자를 주입한다:
   - `arrayCtor`: `SignalArray as unknown as ArrayConstructor`
   - `mapCtor`: `SignalMap as unknown as MapConstructor`
   - `objCtor`: `SignalObject as unknown as ObjectConstructor`
   - `setCtor`: `SignalSet as unknown as SetConstructor`
3. 각 Signal 컬렉션 타입은 타입 시스템 호환을 위해 `as unknown as <내장타입생성자>` 이중 캐스팅을 사용한다.
4. 생성된 인스턴스를 그대로 반환한다.

**경계 케이스**: 타입 캐스팅은 런타임에는 영향이 없으며, `A2uiMessageProcessor`의 컬렉션 생성자 인터페이스와 Signal 컬렉션 클래스 간의 구조적 호환성에 의존한다.

## 동작 흐름

소비자(`index.ts`)가 `createSignalA2uiMessageProcessor`로 임포트하여 호출하면, Signal 컬렉션이 주입된 `A2uiMessageProcessor` 인스턴스가 반환된다. 이 인스턴스는 내부 데이터 구조 변경 시 Signal 시스템을 통해 자동으로 Lit 컴포넌트 업데이트를 트리거한다.
