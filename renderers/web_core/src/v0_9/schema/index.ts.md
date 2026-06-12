# renderers/web_core/src/v0_9/schema/index.ts

## 개요

`v0_9/schema` 디렉토리의 공개 진입점(barrel file)으로, 하위 4개 모듈의 모든 export를 재내보낸다. 외부 코드가 개별 파일 경로를 알 필요 없이 `schema/index.js` 하나를 import하여 스키마 전체를 사용할 수 있게 한다.

## 의존성

### 저장소 내부 모듈
- [`./common-types.js`](./common-types.ts.md)
- [`./server-to-client.js`](./server-to-client.ts.md)
- [`./client-capabilities.js`](./client-capabilities.ts.md)
- [`./client-to-server.js`](./client-to-server.ts.md)

## Exports

위 4개 파일의 모든 export를 그대로 재내보낸다 (`export * from ...`).

## 동작 흐름

런타임 로직 없음. 모듈 시스템이 import 시 각 파일을 순서대로 평가하며, 이름 충돌이 없는 한 모든 export가 단일 네임스페이스로 합쳐진다.
