# renderers/react/src/v0_9/index.ts

## 개요

v0_9 렌더러 패키지의 최상위 진입점(barrel) 파일이다. `A2uiSurface`, `adapter`, basic catalog 전체를 한 번에 재수출하여, 외부 소비자가 단일 import 경로로 모든 v0_9 공개 API에 접근할 수 있도록 한다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈
- `./A2uiSurface` — `export *` 재수출
- [`./adapter`](./adapter.tsx.md) — `export *` 재수출
- [`./catalog/basic`](./catalog/basic/index.ts.md) — `export *` 재수출

## Exports

`export *` 형태로 아래 모듈의 모든 공개 항목을 재수출한다:

| 출처 | 주요 항목 |
|------|-----------|
| `./A2uiSurface` | `A2uiSurface` 컴포넌트 및 관련 타입 |
| `./adapter` | `ReactComponentImplementation`, `ReactA2uiComponentProps`, `createComponentImplementation`, `createBinderlessComponentImplementation` |
| `./catalog/basic` | `basicCatalog`, 18개 컴포넌트 구현체, `MarkdownContext`, `useMarkdownRenderer` |

## 동작 흐름

모듈 로드 시 세 하위 모듈이 모두 초기화되며, 특히 `./catalog/basic`이 로드될 때 `basicCatalog` 인스턴스가 생성된다. 이후 패키지 소비자는 이 파일 하나에서 v0_9의 모든 공개 API를 import할 수 있다.
