# renderers/react/a2ui_explorer/tests/setup.ts

## 개요

Karma + 브라우저 기반 통합 테스트 환경 설정 파일이다. React 18의 `act()` 테스트 환경 플래그인 `globalThis.IS_REACT_ACT_ENVIRONMENT`를 `true`로 설정하여, 상태 전환 및 마운트 과정에서 발생하는 React의 콘솔 경고를 억제한다.

## 의존성

없음 (TypeScript 전역 선언만 포함)

## Exports

없음

## 상세 명세

### 전역 타입 선언

`declare global` 블록에서 `IS_REACT_ACT_ENVIRONMENT: boolean | undefined`를 전역 변수로 선언한다. 이는 TypeScript 컴파일러가 해당 전역 변수를 인식할 수 있도록 하는 타입 확장이다.

### 전역 플래그 설정

`globalThis.IS_REACT_ACT_ENVIRONMENT = true`로 설정한다. 이 플래그가 있으면 React는 `act()` 블록 없이 상태 업데이트가 발생할 때 경고를 출력하지 않고 예외를 발생시키는 엄격 모드로 동작한다.

## 동작 흐름

Karma 테스트 러너가 테스트 파일보다 먼저 이 파일을 로드하며, 파일이 로드되는 순간 전역 플래그가 설정된다.
