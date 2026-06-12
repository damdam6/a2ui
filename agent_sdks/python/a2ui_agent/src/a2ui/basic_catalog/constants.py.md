# agent_sdks/python/a2ui_agent/src/a2ui/basic_catalog/constants.py

## 개요

기본(basic) A2UI 카탈로그와 관련된 상수를 정의하는 모듈이다. 카탈로그의 이름과 각 스펙 버전별 카탈로그 스키마 파일의 저장소 내 상대 경로를 매핑한다. 빌드 훅과 런타임 카탈로그 로더 양쪽에서 참조된다.

## 의존성

### 저장소 내부 모듈
- `..schema.constants` — `CATALOG_SCHEMA_KEY`, `VERSION_0_8`, `VERSION_0_9`, `VERSION_0_9_1` 상수

## Exports

- `BASIC_CATALOG_NAME` (상수)
- `BASIC_CATALOG_PATHS` (상수)

## 상세 명세

### `BASIC_CATALOG_NAME: str = "basic"`

기본 카탈로그의 이름 문자열.

### `BASIC_CATALOG_PATHS: dict`

버전별 카탈로그 스키마 파일의 저장소 루트 기준 상대 경로 매핑. 구조는 `{version: {CATALOG_SCHEMA_KEY: relative_path}}`.

| 버전 키 (`VERSION_*`) | `CATALOG_SCHEMA_KEY` 값 |
|---|---|
| `VERSION_0_8` | `"specification/v0_8/json/standard_catalog_definition.json"` |
| `VERSION_0_9` | `"specification/v0_9/catalogs/basic/catalog.json"` |
| `VERSION_0_9_1` | `"specification/v0_9_1/catalogs/basic/catalog.json"` |

## 동작 흐름

빌드 훅(`PackSpecsBuildHook`)에서 이 상수를 읽어 저장소의 스키마 파일을 패키지 에셋으로 복사한다. 런타임에서는 `BundledCatalogProvider`가 `BASIC_CATALOG_PATHS`를 참조해 번들된 리소스를 로드한다.
