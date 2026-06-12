# renderers/angular/a2ui_explorer/src/app/types.ts

## 개요

a2ui_explorer 애플리케이션 전체에서 사용하는 핵심 타입과 토큰 re-export를 정의하는 파일이다. 지원하는 A2UI 프로토콜 버전을 열거형으로 선언하고, 버전별 예제 구성 인터페이스를 정의한다. `A2UI_VERSION`과 `A2UI_EXAMPLES` DI 토큰을 외부로 re-export하여 소비자가 이 파일 하나만 import해도 되는 단일 진입점 역할을 한다.

## 의존성

### 외부 패키지
- `@a2ui/web_core/v0_9` — `A2uiMessage` 타입
- `src/v0_8/types` — `ServerToClientMessage` 타입

### 저장소 내부 모듈
- [`./version_injector`](./version_injector.ts.md) — `A2UI_VERSION` (re-export)
- `./examples_injector` — `A2UI_EXAMPLES` (re-export)

## Exports

| 이름 | 종류 | 설명 |
|---|---|---|
| `Version` | enum | 지원하는 프로토콜 버전 열거형 |
| `A2uiExample` | 타입 별칭 | `Example \| Example_08` 유니온 타입 |
| `Example` | interface | v0.9 예제 구성 인터페이스 |
| `Example_08` | interface | v0.8 예제 구성 인터페이스 |
| `A2UI_VERSION` | InjectionToken (re-export) | `./version_injector`에서 re-export |
| `A2UI_EXAMPLES` | InjectionToken (re-export) | `./examples_injector`에서 re-export |

## 상세 명세

### `enum Version`

지원하는 A2UI 프로토콜 버전을 나타내는 열거형이다.

| 멤버 | 값 |
|---|---|
| `V0_8` | `'v0.8'` |
| `V0_9` | `'v0.9'` |

### `type A2uiExample`

`Example | Example_08` 유니온 타입이다. 버전에 무관하게 예제를 다루는 코드에서 사용되며, `version` 리터럴 필드가 타입 가드 역할을 한다.

### `interface Example`

v0.9 예제 구성을 나타내는 인터페이스다.

| 필드 | 타입 | 설명 |
|---|---|---|
| `version` | `'0.9'` (리터럴) | 버전 식별자 |
| `name` | `string` | 사이드바에 표시되는 예제 이름 |
| `description` | `string` | 예제가 시연하는 기능에 대한 짧은 설명 |
| `messages` | `A2uiMessage[]` | 렌더러에 전송할 A2UI 메시지 시퀀스 |

### `interface Example_08`

v0.8 예제 구성을 나타내는 인터페이스다. `Example`과 구조는 동일하되 `version` 리터럴과 `messages` 타입만 다르다.

| 필드 | 타입 | 설명 |
|---|---|---|
| `version` | `'0.8'` (리터럴) | 버전 식별자 |
| `name` | `string` | 사이드바에 표시되는 예제 이름 |
| `description` | `string` | 예제가 시연하는 기능에 대한 짧은 설명 |
| `messages` | `ServerToClientMessage[]` | 렌더러에 전송할 v0.8 메시지 시퀀스 |

## 동작 흐름

이 파일은 런타임 로직이 없다. 열거형·인터페이스 선언과 두 개의 re-export로만 구성되어, 소비자가 버전 상수와 DI 토큰을 단일 위치에서 import할 수 있도록 한다. `Version` 열거형의 리터럴 값(`'v0.8'`, `'v0.9'`)은 URL 쿼리 파라미터 파싱 등 런타임 비교에 사용된다.
