# renderers/react/tests/v0_8/utils/assertions.ts

## 개요

v0.8 테스트에서 공통으로 사용하는 타입 안전 assertion 헬퍼 유틸리티 모듈이다. vitest `Mock` 객체의 호출 인자를 안전하게 추출하거나, 배열의 특정 인덱스 요소를 경계 검사와 함께 가져오는 두 가지 함수를 제공한다. 존재하지 않는 항목에 접근 시 undefined 반환 대신 명시적 에러를 던져 테스트 실패 원인을 명확히 한다.

## 의존성

### 외부 패키지
- `vitest`: `Mock` 타입 (type-only import)

### 저장소 내부 모듈
없음.

## Exports

| 이름 | 종류 |
|---|---|
| `getMockCallArg` | 함수 (제네릭) |
| `getElement` | 함수 (제네릭) |

## 상세 명세

### `getMockCallArg<T>(mock: Mock, callIndex: number, argIndex?: number): T`

- **매개변수**
  - `mock: Mock` — vitest의 Mock 객체
  - `callIndex: number` — 호출 순서 인덱스 (0부터 시작)
  - `argIndex: number` — 해당 호출에서 가져올 인자 인덱스, 기본값 `0`
- **반환 타입**: 제네릭 `T` (호출자가 지정)
- **동작 로직**
  1. `mock.mock.calls` 배열에서 `callIndex` 위치의 호출을 꺼낸다.
  2. 해당 호출이 존재하지 않으면(`!call`) 에러를 던진다. 에러 메시지: `` `Mock call at index ${callIndex} does not exist. Total calls: ${calls.length}` ``
  3. 존재하면 `call[argIndex]`를 `T`로 캐스팅하여 반환한다.

### `getElement<T>(array: T[], index: number): T`

- **매개변수**
  - `array: T[]` — 임의의 배열
  - `index: number` — 가져올 인덱스
- **반환 타입**: 제네릭 `T`
- **동작 로직**
  1. `array[index]`를 꺼낸다.
  2. 결과가 `undefined`이면 에러를 던진다. 에러 메시지: `` `Array element at index ${index} does not exist. Array length: ${array.length}` ``
  3. 정상이면 해당 요소를 반환한다.

## 동작 흐름

두 함수 모두 단일 책임 헬퍼다. 호출자가 특정 타입 `T`를 지정하면, 경계 검사 후 타입 단언(`as T`)으로 해당 타입으로 반환한다. undefined 처리를 런타임 에러로 바꾸어 테스트에서 옵셔널 체이닝 없이 깔끔하게 사용할 수 있도록 설계되었다.
