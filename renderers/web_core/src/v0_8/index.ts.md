# renderers/web_core/src/v0_8/index.ts

## 개요

`v0_8` 패키지의 공개 API 진입점이다. 데이터 처리, 스타일, 타입, 에러, 이벤트 관련 모듈을 한곳에서 재수출하고, JSON 스키마 파일을 `Schemas` 네임스페이스 객체로 묶어 내보낸다. 외부 소비자는 이 파일 하나만 임포트하면 라이브러리 전체 공개 API에 접근할 수 있다.

## 의존성

### 외부 패키지
- (없음)

### 저장소 내부 모듈
- [`./events/index.js`](./events/index.ts.md) — `Events` 네임스페이스로 재수출
- [`./data/guards.js`](./data/guards.ts.md) — 모든 공개 guard 함수
- [`./data/model-processor.js`](./data/model-processor.ts.md) — `A2uiMessageProcessor`
- [`./styles/index.js`](./styles/index.ts.md) — 스타일 관련 공개 항목
- [`./types/colors.js`](./types/colors.ts.md) — 색상 타입
- [`./types/primitives.js`](./types/primitives.ts.md) — 기본 원시 타입
- [`./types/types.js`](./types/types.ts.md) — 핵심 타입 정의
- [`./errors.js`](./errors.ts.md) — `A2uiError` 및 파생 에러 클래스
- `./schemas/server_to_client_with_standard_catalog.json` — JSON 스키마 파일 (import assertion `{ type: 'json' }`)

## Exports

| 이름 | 종류 | 출처 |
|------|------|------|
| `Events` | 네임스페이스 (re-export as namespace) | `./events/index.js` |
| (guards 전체) | 함수들 | `./data/guards.js` |
| `A2uiMessageProcessor` | 클래스 | `./data/model-processor.js` |
| (styles 전체) | 다양 | `./styles/index.js` |
| (colors 전체) | 타입/상수 | `./types/colors.js` |
| (primitives 전체) | 타입/상수 | `./types/primitives.js` |
| (types 전체) | 타입/인터페이스 | `./types/types.js` |
| (errors 전체) | 클래스 | `./errors.js` |
| `Schemas` | 상수 객체 | 이 파일에서 직접 정의 |

### `Schemas` 상수

```ts
export const Schemas = {
  A2UIClientEventMessage,
};
```

- `A2UIClientEventMessage` 프로퍼티에 `server_to_client_with_standard_catalog.json` 전체를 JSON 객체로 보유한다.
- 소비자가 런타임에 스키마 정의에 접근하거나 유효성 검사 도구에 전달하기 위한 용도이다.

## 동작 흐름

파일 자체의 실행 코드는 `Schemas` 상수 초기화 하나뿐이다. 나머지는 모두 re-export이므로, 소비자가 `import { A2uiMessageProcessor, Schemas, Events } from '.../v0_8/index.js'`와 같이 단일 경로로 필요한 항목을 임포트한다.
