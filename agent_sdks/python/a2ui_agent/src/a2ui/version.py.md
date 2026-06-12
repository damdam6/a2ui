# agent_sdks/python/a2ui_agent/src/a2ui/version.py

## 개요

`a2ui` 패키지의 현재 버전을 단일 상수로 선언하는 파일이다. 파일 기반 정적 버전 관리 방식을 채택하며, 향후 `hatch-vcs` 플러그인을 통한 Git 태그 기반 VCS 버전 관리로 전환할 계획을 주석으로 명시하고 있다.

## 의존성

없음.

## Exports

- `__version__` (상수, `str`)

## 상세 명세

### `__version__: str`

값: `"0.2.4"`. 형식은 `major.minor.patch`. 릴리스 전 이 파일에서 직접 갱신해야 한다.

## 동작 흐름

모듈 임포트 시 `__version__ = "0.2.4"`가 평가된다. 다른 모듈에서 `from a2ui.version import __version__`으로 참조한다.
