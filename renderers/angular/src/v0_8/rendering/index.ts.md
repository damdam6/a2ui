# renderers/angular/src/v0_8/rendering/index.ts

## 개요

`rendering` 서브패키지의 배럴(barrel) 파일로, 렌더링 인프라를 구성하는 네 모듈(`catalog`, `dynamic-component`, `renderer`, `theming`)의 모든 공개 심볼을 하나의 경로에서 임포트할 수 있도록 재내보낸다.

## 의존성

### 저장소 내부 모듈

- [`./catalog`](catalog.ts.md) — `CatalogLoader`, `CatalogEntry`, `Catalog` (인터페이스 + 토큰)
- [`./dynamic-component`](dynamic-component.ts.md) — `DynamicComponent`
- [`./renderer`](renderer.ts.md) — `Renderer`
- [`./theming`](theming.ts.md) — `Theme`

### 외부 패키지

없음

## Exports

각 모듈의 공개 심볼 전체를 flat하게 재내보낸다:

| 원본 모듈 | 재내보내는 심볼 |
|---|---|
| `./catalog` | `CatalogLoader`, `CatalogEntry<T>`, `Catalog` (인터페이스 및 토큰) |
| `./dynamic-component` | `DynamicComponent<T>` |
| `./renderer` | `Renderer` |
| `./theming` | `Theme` |

## 동작 흐름

런타임 로직 없음. TypeScript 컴파일 시 각 `export *` 구문이 처리되어 소비자가 `rendering/index` 하나만 임포트해도 모든 렌더링 심볼에 접근할 수 있다.
