# agent_sdks/python/a2ui_agent/src/a2ui/__init__.py

## 개요

`a2ui` 패키지의 최상위 초기화 모듈이다. 패키지 버전 정보를 외부에 노출하는 것이 유일한 역할이다. 별도의 로직 없이 `version` 서브모듈에서 `__version__`을 재내보낸다.

## 의존성

### 저장소 내부 모듈
- `.version` — `__version__` 문자열을 정의하는 동일 패키지 내 모듈 (파일: `agent_sdks/python/a2ui_agent/src/a2ui/version.py`)

## Exports

- `__version__` (문자열 상수) — 패키지의 버전 문자열

## 상세 명세

파일 전체가 `from .version import __version__` 한 줄로 구성된다. `__version__` 값은 `version.py`에서 정의되며, 이 파일은 그것을 패키지 네임스페이스로 끌어올린다.

## 동작 흐름

`import a2ui` 시 Python이 이 파일을 실행하며, `a2ui.__version__`을 통해 버전 정보에 접근할 수 있게 된다.
