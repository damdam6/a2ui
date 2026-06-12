# renderers/lit/src/0.8/index.ts

## 개요

`renderers/lit/src/0.8` 패키지의 공개 API 진입점이다. `@a2ui/web_core`의 핵심 타입·유틸리티·이벤트를 re-export하고, Signal 기반 메시지 프로세서 팩토리와 함께 `Data` 네임스페이스 객체로 묶어 소비자에게 제공한다. 또한 `core.ts`를 통해 로컬 `Events` 네임스페이스도 노출한다.

## 의존성

### 저장소 내부 모듈
- [`./core.ts`](./core.ts.md) — `Events`(이벤트), `UI` 네임스페이스 re-export (core를 통해 간접 노출)
- [`./data/signal-model-processor.ts`](./data/signal-model-processor.ts.md) — `create` 함수 (`createSignalA2uiMessageProcessor` 별칭으로 import)

### 외부 패키지
- `@a2ui/web_core/types/types` — `Types` 네임스페이스
- `@a2ui/web_core/data/guards` — `Guards` 네임스페이스
- `@a2ui/web_core` — `Schemas`, `Events as WebEvents`
- `@a2ui/web_core/styles/index` — `Styles` 네임스페이스
- `@a2ui/web_core/data/model-processor` — `A2uiMessageProcessor` 클래스
- `@a2ui/web_core/types/primitives` — `Primitives` 네임스페이스

## Exports

| 이름 | 종류 | 설명 |
|---|---|---|
| `Events` | 네임스페이스 (re-export) | `./core.ts`를 통해 로컬 이벤트 시스템 (`StateEvent` 등) |
| `Types` | 네임스페이스 | `@a2ui/web_core/types/types` 전체 |
| `Guards` | 네임스페이스 | `@a2ui/web_core/data/guards` 전체 |
| `Schemas` | (named export) | `@a2ui/web_core`의 `Schemas` |
| `Styles` | 네임스페이스 | `@a2ui/web_core/styles/index` 전체 |
| `A2uiMessageProcessor` | 클래스 | 웹 코어 메시지 프로세서 |
| `Primitives` | 네임스페이스 | `@a2ui/web_core/types/primitives` 전체 |
| `WebEvents` | 네임스페이스 | `@a2ui/web_core`의 `Events` (웹 코어 이벤트) |
| `Data` | 상수 객체 | 아래 세 항목을 묶은 네임스페이스 객체 |
| `Data.createSignalA2uiMessageProcessor` | 함수 | Signal 컬렉션 기반 `A2uiMessageProcessor` 팩토리 |
| `Data.A2uiMessageProcessor` | 클래스 | 웹 코어 메시지 프로세서 (중복 노출) |
| `Data.Guards` | 네임스페이스 | `@a2ui/web_core/data/guards` (중복 노출) |

## 상세 명세

### `export * as Events from './events/events.js'`

`core.ts`의 `export * as Events`를 통해 로컬 이벤트 클래스(`StateEvent`, `StateEventDetailMap` 등)를 `Events` 네임스페이스로 노출한다. `index.ts`에서 직접 `from './core.js'`를 통해 재-export된다.

### `export const Data`

**타입**: `{ createSignalA2uiMessageProcessor: () => A2uiMessageProcessor, A2uiMessageProcessor: typeof A2uiMessageProcessor, Guards: typeof Guards }`

데이터 처리 관련 기능을 단일 객체로 묶어 소비자가 `Data.createSignalA2uiMessageProcessor()` 형태로 쉽게 접근할 수 있도록 한다. `A2uiMessageProcessor`와 `Guards`가 함께 포함되어 소비자가 별도 import 없이 데이터 레이어 전체에 접근할 수 있다.

## 동작 흐름

1. 파일 로드 시 모든 import가 해결되고 `Data` 상수 객체가 초기화된다.
2. 소비자는 이 파일을 단일 진입점으로 사용하여 타입, 가드, 스키마, 스타일, 이벤트, 데이터 처리 기능을 일괄 import한다.
3. Signal 기반 렌더링이 필요한 경우 `Data.createSignalA2uiMessageProcessor()`를 호출하여 반응형 메시지 프로세서를 생성한다.
4. 로컬 이벤트(`Events.StateEvent`)와 웹 코어 이벤트(`WebEvents`)가 별도 네임스페이스로 분리되어 혼동 없이 사용할 수 있다.
