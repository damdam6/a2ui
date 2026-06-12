# renderers/lit/src/0.8/core.ts

## 개요

`renderers/lit/src/0.8` 패키지의 핵심 진입 모듈로, 이벤트 시스템과 UI 서브모듈을 통합하여 재-export한다. `./events/events.js`의 모든 내보내기를 `Events` 네임스페이스로, `./ui/ui.js`의 모든 내보내기를 `UI` 네임스페이스로 노출한다.

## 의존성

### 저장소 내부 모듈
- [`./events/events.ts`](./events/events.ts.md) — `StateEvent`, `StateEventDetailMap` 등 이벤트 클래스·타입
- `./ui/ui.js` — UI 컴포넌트 모듈 (이 목록의 분석 대상 외 파일)

### 외부 패키지
없음

## Exports

- `* as Events from './events/events.js'` — 이벤트 관련 모든 심볼을 `Events` 네임스페이스로 재-export
- `* as UI from './ui/ui.js'` — UI 컴포넌트 관련 모든 심볼을 `UI` 네임스페이스로 재-export

## 동작 흐름

이 파일 자체에는 런타임 로직이 없다. 두 서브모듈의 심볼을 네임스페이스 별칭으로 묶어 소비자(예: `index.ts`)가 단일 import 경로로 접근할 수 있도록 중계하는 역할만 한다.
