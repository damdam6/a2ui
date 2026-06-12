# agent_sdks/python/a2ui_agent/src/a2ui/basic_catalog/__init__.py

## 개요

`a2ui.basic_catalog` 서브패키지의 초기화 파일이다. `provider` 모듈에서 `BasicCatalog` 클래스를 재내보내어 패키지 사용자가 `from a2ui.basic_catalog import BasicCatalog`로 접근할 수 있도록 한다.

## 의존성

### 저장소 내부 모듈
- [`.provider`](./provider.py.md) — `BasicCatalog` 클래스 정의

## Exports

- `BasicCatalog` (클래스) — 기본 A2UI 카탈로그 접근 헬퍼

## 동작 흐름

단일 re-export(`from .provider import BasicCatalog`)만 수행한다.
