# agent_sdks/python/a2ui_core/src/a2ui/core/schema/constants.py

## 개요

A2UI 스키마와 메시지 프로토콜에서 사용하는 모든 문자열 상수를 한 곳에 모아 정의한다. 다른 모듈들이 하드코딩 없이 이 상수를 참조해 명세 버전, 메시지 타입, JSON 키 이름을 일관되게 사용할 수 있게 한다.

## 의존성

없음. 표준 라이브러리도 import하지 않는다.

## Exports

| 상수명 | 값 | 설명 |
|---|---|---|
| `SPEC_VERSION` | `"v0.9"` | A2UI 명세 버전 식별자. 메시지의 `version` 필드 alias로도 사용. |
| `SPEC_BASE_URL` | `"https://a2ui.org/specification"` | 명세 문서의 기반 URL. |
| `MSG_TYPE_CREATE_SURFACE` | `"createSurface"` | 서피스 생성 메시지 타입 키. |
| `MSG_TYPE_UPDATE_COMPONENTS` | `"updateComponents"` | 컴포넌트 업데이트 메시지 타입 키. |
| `MSG_TYPE_UPDATE_DATA_MODEL` | `"updateDataModel"` | 데이터 모델 업데이트 메시지 타입 키. |
| `MSG_TYPE_DELETE_SURFACE` | `"deleteSurface"` | 서피스 삭제 메시지 타입 키. |
| `CATALOG_COMPONENTS_KEY` | `"components"` | 카탈로그 JSON 내 컴포넌트 맵의 키 이름. `JsonCatalog.extract_ref_fields`에서 사용. |
| `SURFACE_ID_KEY` | `"surfaceId"` | 서피스 ID를 가리키는 JSON 키 이름. |
| `THEME_KEY` | `"theme"` | 테마 정보를 가리키는 JSON 키 이름. |
| `ROOT_ID` | `"root"` | 루트 컴포넌트/서피스의 기본 ID. |

## 동작 흐름

파일에는 오직 상수 할당문만 존재한다. `schema/__init__.py`에서 `from .constants import *`로 전체를 재노출하므로, `from a2ui.core.schema import SPEC_VERSION`처럼 직접 접근할 수 있다.
