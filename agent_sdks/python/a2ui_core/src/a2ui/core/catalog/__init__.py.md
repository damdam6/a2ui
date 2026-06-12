# agent_sdks/python/a2ui_core/src/a2ui/core/catalog/__init__.py

## 개요

`catalog` 패키지의 공개 API를 하나의 네임스페이스로 집약해 외부에서 편리하게 import할 수 있도록 한다. 하위 모듈 네 개(`catalog`, `model_catalog`, `json_catalog`, `functions`)에서 핵심 심볼을 선별해 재노출하며 별도 로직은 포함하지 않는다.

## 의존성

### 저장소 내부 모듈
- [`./catalog.py`](catalog.py.md) — `CatalogApi`, `CatalogImplementation`
- [`./model_catalog.py`](model_catalog.py.md) — `ModelCatalog`
- [`./json_catalog.py`](json_catalog.py.md) — `JsonCatalog`
- [`./functions.py`](functions.py.md) — `FunctionApi`, `FunctionImplementation`, `FunctionInvoker`, `create_function_implementation`

## Exports

`__all__`에 명시된 공개 심볼:

| 이름 | 종류 |
|---|---|
| `CatalogApi` | 클래스 (추상 기반) |
| `CatalogImplementation` | 클래스 (추상 확장) |
| `ModelCatalog` | 클래스 (Pydantic 기반 구체 구현) |
| `JsonCatalog` | 클래스 (JSON 기반 구체 구현) |
| `FunctionApi` | 클래스 (함수 명세) |
| `FunctionImplementation` | 클래스 (함수 명세 + 실행 구현) |
| `FunctionInvoker` | 타입 별칭 (`Callable`) |
| `create_function_implementation` | 함수 (동적 구성 헬퍼) |

## 동작 흐름

파일은 import 선언과 `__all__` 목록으로만 구성된다. 패키지 사용자는 `from a2ui.core.catalog import ModelCatalog` 형태로 직접 가져올 수 있으며, 개별 하위 모듈 경로를 알 필요가 없다.
