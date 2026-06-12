# renderers/web_core/src/v0_9/basic_catalog/index.ts

## 개요

`basic_catalog` 하위 모듈의 공개 진입점(barrel) 파일이다. 표현식 파서, 기본 함수 구현체, 기본 함수 API 디스크립터, 기본 컴포넌트, 그리고 스타일 관련 유틸리티를 한 곳에서 재내보낸다(re-export). 이 파일 자체에는 로직이 없으며, 하위 모듈들을 외부에서 단일 경로로 가져올 수 있게 집약하는 역할만 한다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈

- [`./expressions/expression_parser.js`](./expressions/expression_parser.ts.md) — 표현식 파서
- [`./functions/basic_functions.js`](./functions/basic_functions.ts.md) — 기본 함수 구현체
- [`./functions/basic_functions_api.js`](./functions/basic_functions_api.ts.md) — 기본 함수 API 디스크립터
- [`./components/basic_components.js`](./components/basic_components.ts.md) — 기본 컴포넌트 정의
- [`./styles/default.js`](./styles/default.ts.md) — 기본 테마 스타일 유틸리티

## Exports

모든 항목은 하위 모듈에서 위임(re-export)한 것이다.

| 출처 | 내보내는 항목 |
|---|---|
| `./expressions/expression_parser.js` | 해당 모듈의 모든 공개 export |
| `./functions/basic_functions.js` | 해당 모듈의 모든 공개 export |
| `./functions/basic_functions_api.js` | 해당 모듈의 모든 공개 export |
| `./components/basic_components.js` | 해당 모듈의 모든 공개 export |
| `./styles/default.js` | `injectBasicCatalogStyles`(함수), `computeColorVariant`(함수) |
| `./styles/default.js` (타입) | `ColorVariantLightDarkOptions`(인터페이스), `ColorVariantHoverOptions`(인터페이스) |

## 동작 흐름

파일이 임포트되면 각 하위 모듈이 로드되며, 소비자는 이 단일 경로(`@a2ui/web-core/v0_9/basic_catalog` 또는 상대 경로)를 통해 기본 카탈로그의 모든 공개 API에 접근한다. 자체 실행 코드는 없다.
