# renderers/angular/src/v0_8/data/index.ts

## 개요

`data` 디렉토리의 공개 API를 집약하는 배럴(barrel) 파일이다. `processor`, `types`, `markdown` 세 모듈의 내보내기를 재노출하여 외부에서 단일 경로(`../data` 또는 `./data`)로 접근할 수 있도록 한다.

## 의존성

### 저장소 내부 모듈
- [`./processor`](./processor.ts.md) — `MessageProcessor`, `A2UIClientEvent`, `DispatchedEvent`
- [`./types`](./types.ts.md) — `A2TextPayload`, `A2DataPayload`, `A2AServerPayload`
- [`./markdown`](./markdown.ts.md) — `provideMarkdownRenderer` (named re-export)

## Exports

| 이름 | 출처 |
|---|---|
| `./processor`의 모든 공개 심볼 | `export * from './processor'` |
| `./types`의 모든 공개 심볼 | `export * from './types'` |
| `provideMarkdownRenderer` | `export {provideMarkdownRenderer} from './markdown'` |

## 동작 흐름

이 파일은 실행 로직 없이 재내보내기(re-export)만 수행한다. `markdown` 모듈에서는 `provideMarkdownRenderer`만 선택적으로 노출하고 `MarkdownRenderer`, `DefaultMarkdownRenderer`는 직접 노출하지 않는다.
