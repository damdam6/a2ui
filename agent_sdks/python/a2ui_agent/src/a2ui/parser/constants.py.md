# agent_sdks/python/a2ui_agent/src/a2ui/parser/constants.py

## 개요

A2UI 파싱에 사용되는 메시지 타입 문자열 상수를 정의하는 모듈이다. v0.8과 v0.9 두 프로토콜 버전의 메시지 타입 식별자, 기본 루트 ID, 비-A2UI 텍스트 타입 상수를 제공한다. 다른 파서 모듈들이 이 파일을 `from .constants import *` 형태로 임포트하여 사용한다.

## 의존성

- 외부 패키지: 없음
- 내부 모듈: 없음

## Exports

모든 상수가 모듈 레벨에서 공개된다:

- `DEFAULT_ROOT_ID` — 상수 (str)
- `MSG_TYPE_BEGIN_RENDERING` — 상수 (str)
- `MSG_TYPE_SURFACE_UPDATE` — 상수 (str)
- `MSG_TYPE_DATA_MODEL_UPDATE` — 상수 (str)
- `MSG_TYPE_DELETE_SURFACE` — 상수 (str)
- `MSG_TYPE_CREATE_SURFACE` — 상수 (str)
- `MSG_TYPE_UPDATE_COMPONENTS` — 상수 (str)
- `MSG_TYPE_UPDATE_DATA_MODEL` — 상수 (str)
- `MSG_TYPE_TEXT` — 상수 (str)

## 상세 명세

### `DEFAULT_ROOT_ID`
- 값: `"root"`
- 서피스의 루트 컴포넌트 ID 기본값이다.

### v0.8 메시지 타입

| 상수 | 값 |
|------|----|
| `MSG_TYPE_BEGIN_RENDERING` | `"beginRendering"` |
| `MSG_TYPE_SURFACE_UPDATE` | `"surfaceUpdate"` |
| `MSG_TYPE_DATA_MODEL_UPDATE` | `"dataModelUpdate"` |
| `MSG_TYPE_DELETE_SURFACE` | `"deleteSurface"` |

### v0.9 메시지 타입

| 상수 | 값 |
|------|----|
| `MSG_TYPE_CREATE_SURFACE` | `"createSurface"` |
| `MSG_TYPE_UPDATE_COMPONENTS` | `"updateComponents"` |
| `MSG_TYPE_UPDATE_DATA_MODEL` | `"updateDataModel"` |

`MSG_TYPE_DELETE_SURFACE` (`"deleteSurface"`)는 v0.8과 v0.9가 공유한다.

### `MSG_TYPE_TEXT`
- 값: `"text"`
- A2UI 구조가 아닌 일반 대화형 텍스트 메시지를 나타낸다.

## 동작 흐름

모듈 임포트 시 모든 상수가 즉시 모듈 네임스페이스에 바인딩된다. 실행 로직은 없다.
