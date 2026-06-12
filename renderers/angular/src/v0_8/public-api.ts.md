# renderers/angular/src/v0_8/public-api.ts

## 개요

v0.8 Angular 렌더러의 공개 API 진입점 파일이다. 이 파일은 `v0_8` 하위 패키지가 외부 소비자에게 노출하는 모든 심볼을 한곳에서 재내보낸다. 개별 컴포넌트, 데이터 처리, 렌더링 인프라, 타입 정의를 모두 하나의 엔트리 포인트로 집약하여, 소비자가 단일 경로(`@a2ui/angular/v0_8` 등)에서 필요한 모든 심볼을 임포트할 수 있게 한다.

## 의존성

### 저장소 내부 모듈

- [`./catalog/index`](catalog/index.ts.md) — 기본 카탈로그(`DEFAULT_CATALOG`) 및 등록 함수
- `./components/audio` — `AudioPlayer` 컴포넌트
- `./components/button` — `Button` 컴포넌트
- `./components/card` — `Card` 컴포넌트
- `./components/checkbox` — `Checkbox` 컴포넌트
- `./components/column` — `Column` 컴포넌트
- `./components/datetime-input` — `DateTimeInput` 컴포넌트
- `./components/divider` — `Divider` 컴포넌트
- `./components/icon` — `Icon` 컴포넌트
- `./components/image` — `Image` 컴포넌트
- `./components/list` — `List` 컴포넌트
- `./components/modal` — `Modal` 컴포넌트
- `./components/multiple-choice` — `MultipleChoice` 컴포넌트
- `./components/row` — `Row` 컴포넌트
- `./components/slider` — `Slider` 컴포넌트
- `./components/surface` — `Surface` 컴포넌트
- `./components/tabs` — `Tabs` 컴포넌트
- `./components/text-field` — `TextField` 컴포넌트
- `./components/text` — `Text` 컴포넌트
- `./components/video` — `Video` 컴포넌트
- `./config` — 설정 관련 심볼
- [`./data/index`](data/index.ts.md) — 데이터 처리 레이어
- [`./rendering/index`](rendering/index.ts.md) — 렌더링 인프라 (Renderer, Catalog, DynamicComponent, Theme)
- [`./types`](types.ts.md) — `Types` 네임스페이스로 묶인 타입 정의

### 외부 패키지

없음 (모두 내부 재내보내기)

## Exports

모든 임포트를 `export *` 또는 `export * as` 형태로 재내보낸다:

- `./catalog/index`, `./components/*`, `./config`, `./data/index`, `./rendering/index` — 각 모듈의 공개 심볼 전체를 flat하게 재내보냄
- `Types` — `./types`의 모든 심볼을 `Types` 네임스페이스 객체로 묶어 내보냄 (`export * as Types from './types'`)

## 상세 명세

### 재내보내기 패턴

이 파일 자체에는 어떤 선언도 없으며, 오직 `export *` 및 `export * as` 구문만 포함된다.

- 컴포넌트 18종(`audio`, `button`, `card`, `checkbox`, `column`, `datetime-input`, `divider`, `icon`, `image`, `list`, `modal`, `multiple-choice`, `row`, `slider`, `surface`, `tabs`, `text-field`, `text`, `video`)이 각각 개별 파일에서 flat 재내보내기된다.
- `./rendering/index`를 통해 `Renderer`, `Catalog`, `CatalogEntry`, `CatalogLoader`, `DynamicComponent`, `Theme`이 노출된다.
- `./data/index`를 통해 `MessageProcessor` 등 데이터 레이어가 노출된다.
- `./types`는 `Types` 네임스페이스 객체로 묶여, 소비자가 `Types.AnyComponentNode` 처럼 접근한다.

## 동작 흐름

파일이 로드되면 TypeScript 컴파일러가 각 `export *` 구문을 처리하여 해당 모듈의 모든 공개 심볼을 이 모듈의 공개 인터페이스로 등록한다. 런타임 로직은 없고, 순수한 정적 재내보내기 매핑만 존재한다.
