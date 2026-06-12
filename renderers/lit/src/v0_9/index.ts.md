# renderers/lit/src/v0_9/index.ts

## 개요

`@a2ui/lit/v0_9` 패키지의 공개 API 진입점 파일이다. 타입, 컨트롤러, 서피스 엘리먼트, 베이스 엘리먼트, 컨텍스트, 기본 카탈로그를 단일 지점에서 re-export하여 소비자가 하나의 임포트 경로로 패키지의 모든 핵심 기능에 접근할 수 있게 한다.

## 의존성

### 저장소 내부 모듈
- [`./types.js`](./types.ts.md) — `LitComponentApi` 타입
- [`./a2ui-controller.js`](./a2ui-controller.ts.md) — `A2uiController` 클래스
- [`./surface/a2ui-surface.js`](./surface/a2ui-surface.ts.md) — `A2uiSurface` 커스텀 엘리먼트
- [`./a2ui-lit-element.js`](./a2ui-lit-element.ts.md) — `A2uiLitElement` 베이스 클래스
- [`./context/context.js`](./context/context.ts.md) — `Context` 네임스페이스 객체
- [`./catalogs/basic/index.js`](./catalogs/basic/index.ts.md) — `basicCatalog` 인스턴스

## Exports

- `LitComponentApi` (타입): Lit 컴포넌트 API 인터페이스 타입 (type-only re-export)
- `A2uiController` (클래스): A2UI 컴포넌트 컨트롤러
- `A2uiSurface` (클래스): A2UI 서피스 커스텀 엘리먼트
- `A2uiLitElement` (클래스): A2UI Lit 컴포넌트 베이스 클래스
- `Context` (상수 객체): 마크다운 렌더러 등 의존성 주입 컨텍스트 모음
- `basicCatalog` (상수): Basic Catalog `Catalog<LitComponentApi>` 인스턴스

## 동작 흐름

단순 배럴(barrel) 파일로 자체 로직이 없다. 소비자는 `import { A2uiSurface, basicCatalog, Context, ... } from '@a2ui/lit/v0_9'`와 같이 하나의 경로로 패키지 공개 API 전체에 접근한다.
