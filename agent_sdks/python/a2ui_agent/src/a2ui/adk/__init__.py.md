# agent_sdks/python/a2ui_agent/src/a2ui/adk/__init__.py

## 개요

`a2ui.adk` 서브패키지의 초기화 파일이다. 현재 아무런 심볼도 export하지 않으며, ADK(Agent Development Kit) 통합 관련 하위 모듈들의 패키지 네임스페이스 역할만 한다.

## 의존성

없음.

## Exports

없음 (빈 파일, 라이선스 주석만 존재).

## 동작 흐름

`import a2ui.adk` 시 이 파일이 실행되어 패키지 디렉토리를 Python 패키지로 인식시킨다. 실제 기능은 하위 모듈(`a2ui.adk.a2a`, `a2ui.adk.send_a2ui_to_client_toolset` 등)에 위치한다.
