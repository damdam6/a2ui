# agent_sdks/kotlin/src/main/kotlin/com/google/a2ui/schema/A2uiConstants.kt

## 개요

A2UI 프로토콜 전반에 걸쳐 사용되는 문자열 상수와 집합을 한 곳에 집약한 싱글턴 객체다. Python SDK와 값 일치를 유지하도록 설계되었으며, 스키마 키 이름, 버전 문자열, 프로토콜 태그, LLM 워크플로우 규칙 등을 포함한다. 의존 파일이 없는 독립 상수 파일이다.

## 의존성

### 외부 패키지
없음.

### 저장소 내부 모듈
없음.

## Exports

| 이름 | 종류 |
|---|---|
| `A2uiConstants` | object (싱글턴) |

## 상세 명세

### object `A2uiConstants`

패키지 `com.google.a2ui.schema`에 위치하는 `object` 선언이다. 모든 멤버는 상수(`const val`) 또는 JVM 필드(`@JvmField val`)다.

#### 스키마 키 상수

| 상수명 | 값 | 설명 |
|---|---|---|
| `A2UI_ASSET_PACKAGE` | `"com.google.a2ui.assets"` | 번들 에셋 패키지 식별자 |
| `SERVER_TO_CLIENT_SCHEMA_KEY` | `"server_to_client"` | 서버→클라이언트 스키마 파일 키 |
| `COMMON_TYPES_SCHEMA_KEY` | `"common_types"` | 공통 타입 스키마 파일 키 |
| `CATALOG_SCHEMA_KEY` | `"catalog"` | 카탈로그 스키마 키 |
| `CATALOG_COMPONENTS_KEY` | `"components"` | 카탈로그 내 컴포넌트 목록 키 |
| `CATALOG_ID_KEY` | `"catalogId"` | 카탈로그 고유 식별자 키 |
| `CATALOG_STYLES_KEY` | `"styles"` | 카탈로그 스타일 키 |

#### 커팅 가능 키 집합

`DEFAULT_CUTTABLE_KEYS`는 `@JvmField val`로 선언된 `Set<String>`이며 값은 `setOf("literalString", "valueString", "label", "hint", "caption", "altText", "text")`다. 스트리밍 중 텍스트 컨텐츠를 점진적으로 방출할 수 있는 JSON 프로퍼티 이름들이다. 문자열 값이 아직 완성되지 않아도 일부를 내보낼 수 있는 필드를 의미한다.

#### 프로토콜 협상 상수

| 상수명 | 값 |
|---|---|
| `SUPPORTED_CATALOG_IDS_KEY` | `"supportedCatalogIds"` |
| `INLINE_CATALOGS_KEY` | `"inlineCatalogs"` |
| `ACCEPTS_INLINE_CATALOGS_KEY` | `"acceptsInlineCatalogs"` |
| `A2UI_CLIENT_CAPABILITIES_KEY` | `"a2uiClientCapabilities"` |

#### 스키마 URL 및 카탈로그 이름

| 상수명 | 값 |
|---|---|
| `BASE_SCHEMA_URL` | `"https://a2ui.org/"` |
| `INLINE_CATALOG_NAME` | `"inline"` |

#### 버전 문자열

| 상수명 | 값 |
|---|---|
| `VERSION_0_8` | `"0.8"` |
| `VERSION_0_9` | `"0.9"` |
| `VERSION_0_9_1` | `"0.9.1"` |

#### XML 래퍼 태그

| 상수명 | 값 |
|---|---|
| `A2UI_OPEN_TAG` | `"<a2ui-json>"` |
| `A2UI_CLOSE_TAG` | `"</a2ui-json>"` |

LLM 응답에서 A2UI JSON 블록의 시작/끝을 표시하는 태그다.

#### 스키마 블록 구분자

| 상수명 | 값 |
|---|---|
| `A2UI_SCHEMA_BLOCK_START` | `"---BEGIN A2UI JSON SCHEMA---"` |
| `A2UI_SCHEMA_BLOCK_END` | `"---END A2UI JSON SCHEMA---"` |

시스템 프롬프트 내 JSON 스키마 섹션의 시작/끝 마커다.

#### `DEFAULT_WORKFLOW_RULES`

`const val`로 선언된 여러 줄 문자열 리터럴이다. LLM에게 전달되는 기본 워크플로우 규칙으로, 다음 네 가지를 명시한다:

1. 응답에 하나 이상의 A2UI JSON 블록이 포함될 수 있다.
2. 각 A2UI JSON 블록은 반드시 `$A2UI_OPEN_TAG`와 `$A2UI_CLOSE_TAG`로 감싸야 한다.
3. 블록 사이 또는 주변에 대화형 텍스트를 넣을 수 있다.
4. JSON 부분은 단일 원시 JSON 객체여야 하며 제공된 A2UI JSON SCHEMA에 대해 유효해야 한다.

문자열 내부에서 `$A2UI_OPEN_TAG`와 `$A2UI_CLOSE_TAG` 상수를 문자열 템플릿으로 참조한다.

## 동작 흐름

이 파일은 동작 로직이 없는 순수 상수 저장소다. 다른 모듈들이 스키마 키 이름, 버전 문자열, 태그 등을 하드코딩하는 대신 이 객체를 참조함으로써 Python SDK와의 값 일치를 보장한다.
