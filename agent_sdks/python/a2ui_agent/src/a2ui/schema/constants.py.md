# agent_sdks/python/a2ui_agent/src/a2ui/schema/constants.py

## 개요

A2UI 스키마 시스템 전반에서 사용되는 문자열 상수와 구성 값들을 중앙 집중적으로 정의하는 모듈이다. 버전 식별자, 스키마 키 이름, 파일 인코딩, 프로토콜 태그, LLM 지시문 블록 마커, 도구(tool) 관련 상수, 그리고 스트리밍 heal 허용 키 집합을 포함한다. 다른 모든 스키마 모듈과 파서 모듈이 이 파일에서 상수를 임포트한다.

## 의존성

없음 (표준 라이브러리 또는 외부 패키지 미사용).

## Exports

모든 이름이 모듈 수준 상수로 공개된다.

### 에셋/스키마 키 상수

- `A2UI_ASSET_PACKAGE: str = "a2ui.assets"` — 번들 리소스 패키지 이름
- `SERVER_TO_CLIENT_SCHEMA_KEY: str = "server_to_client"` — s2c 스키마 딕셔너리 키
- `COMMON_TYPES_SCHEMA_KEY: str = "common_types"` — 공통 타입 스키마 딕셔너리 키
- `CATALOG_SCHEMA_KEY: str = "catalog"` — 카탈로그 스키마 딕셔너리 키
- `CATALOG_COMPONENTS_KEY: str = "components"` — 카탈로그 내 컴포넌트 딕셔너리 키
- `CATALOG_ID_KEY: str = "catalogId"` — 카탈로그 ID 필드 키
- `CATALOG_STYLES_KEY: str = "styles"` — 카탈로그 스타일 필드 키
- `SURFACE_ID_KEY: str = "surfaceId"` — 메시지 내 surface ID 필드 키

### 스트리밍 heal 허용 키

- `DEFAULT_CUTTABLE_KEYS: frozenset[str]` — 스트리밍 중 자동 닫기(heal)가 허용되는 문자열 값 키 집합. 값: `{"literalString", "valueString", "label", "hint", "caption", "altText", "text"}`. `id`, `surfaceId`, `path` 등 구조적·원자적 키는 포함되지 않는다.

### 프로토콜 상수

- `SUPPORTED_CATALOG_IDS_KEY: str = "supportedCatalogIds"`
- `INLINE_CATALOGS_KEY: str = "inlineCatalogs"`
- `A2UI_CLIENT_CAPABILITIES_KEY: str = "a2uiClientCapabilities"`
- `BASE_SCHEMA_URL: str = "https://a2ui.org/"`
- `INLINE_CATALOG_NAME: str = "inline"`

### 버전 상수

- `VERSION_0_8: str = "0.8"`
- `VERSION_0_9: str = "0.9"`
- `VERSION_0_9_1: str = "0.9.1"`

### 버전-스키마 경로 매핑

- `SPEC_VERSION_MAP: dict` — 버전 문자열을 키로, 각 스키마 파일의 리소스 경로를 값으로 갖는 딕셔너리.
  - `"0.8"`: `{"server_to_client": "specification/v0_8/json/server_to_client.json"}`
  - `"0.9"`: s2c `"specification/v0_9/json/server_to_client.json"` + `"common_types": "specification/v0_9/json/common_types.json"`
  - `"0.9.1"`: s2c `"specification/v0_9_1/json/server_to_client.json"` + `"common_types": "specification/v0_9_1/json/common_types.json"`

### 기타 상수

- `SPECIFICATION_DIR: str = "specification"`
- `ENCODING: str = "utf-8"` — 파일 읽기/쓰기 인코딩
- `A2UI_OPEN_TAG: str = "<a2ui-json>"` — LLM 응답 내 A2UI JSON 블록 시작 태그
- `A2UI_CLOSE_TAG: str = "</a2ui-json>"` — LLM 응답 내 A2UI JSON 블록 종료 태그
- `A2UI_SCHEMA_BLOCK_START: str = "---BEGIN A2UI JSON SCHEMA---"` — 스키마 블록 시작 마커
- `A2UI_SCHEMA_BLOCK_END: str = "---END A2UI JSON SCHEMA---"` — 스키마 블록 종료 마커
- `DEFAULT_WORKFLOW_RULES: str` — LLM에 전달하는 기본 워크플로 규칙 문자열. `A2UI_OPEN_TAG`/`A2UI_CLOSE_TAG` 태그 사용, 단일 JSON 객체, 스키마 준수, Top-Down 컴포넌트 순서(`root`가 첫 번째, 부모가 자식보다 먼저) 규칙 등을 명시하는 다행 f-string이다.
- `A2UI_TOOL_NAME: str = "send_a2ui_json_to_client"` — A2UI 도구 이름
- `A2UI_VALIDATED_JSON_KEY: str = "validated_a2ui_json"` — 도구 응답 내 검증된 JSON 키
- `A2UI_TOOL_ERROR_KEY: str = "error"` — 도구 응답 내 오류 키

## 동작 흐름

모듈 임포트 시 모든 상수가 즉시 정의된다. `DEFAULT_WORKFLOW_RULES`는 `A2UI_OPEN_TAG`/`A2UI_CLOSE_TAG`를 f-string으로 참조하므로 해당 상수가 먼저 정의되어야 한다 (파일 내 정의 순서에 의해 보장됨).
