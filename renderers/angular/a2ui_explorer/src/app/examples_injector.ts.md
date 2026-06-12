# renderers/angular/a2ui_explorer/src/app/examples_injector.ts

## 개요

Angular DI 토큰 `A2UI_EXAMPLES`를 정의하고 루트 레벨에서 제공하는 모듈이다. 현재 활성화된 A2UI 프로토콜 버전(`A2UI_VERSION`)을 주입받아, `Version.V0_9`이면 `EXAMPLES_V09`, 그 외이면 `EXAMPLES_V08` 예제 배열을 단일 인스턴스로 반환하는 팩토리를 등록한다.

## 의존성

### 외부 패키지
- `@angular/core`: `InjectionToken`, `inject`

### 저장소 내부 모듈
- `./generated/examples-bundle` (빌드 타임 생성 파일, 소스 저장소에 없음) — `EXAMPLES_V08`, `EXAMPLES_V09`
- [`./types`](./types.ts.md) — `Version`, `A2uiExample`
- [`./version_injector`](./version_injector.ts.md) — `A2UI_VERSION`

## Exports

- `A2UI_EXAMPLES` (상수, `InjectionToken<Array<A2uiExample>>`)

## 상세 명세

### `A2UI_EXAMPLES` 상수

**타입**: `InjectionToken<Array<A2uiExample>>`

`new InjectionToken<Array<A2uiExample>>('A2UI_EXAMPLES', { providedIn: 'root', factory })` 형태로 생성된다.

팩토리 함수는 다음 순서로 동작한다:
1. `inject(A2UI_VERSION)`으로 현재 프로토콜 버전을 가져온다.
2. 버전이 `Version.V0_9`이면 `EXAMPLES_V09`를 반환한다.
3. 그 외의 모든 값(현재는 `Version.V0_8`)이면 `EXAMPLES_V08`을 반환한다.

`providedIn: 'root'`이므로 별도 `providers` 배열 등록 없이 앱 전체에서 주입 가능하다.

## 동작 흐름

DI 시스템이 처음으로 `A2UI_EXAMPLES` 토큰을 요청할 때 팩토리가 한 번 실행되어 버전에 맞는 예제 배열이 생성된다. 이후 같은 인스턴스가 모든 주입 요청에 재사용된다. `DemoComponent`가 `inject(A2UI_EXAMPLES)`로 이 토큰을 사용해 사이드바 예제 목록을 구성한다.
