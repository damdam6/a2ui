# agent_sdks/python/a2ui_core/src/a2ui/core/validating/__init__.py

## 개요

`validating` 패키지의 공개 API를 하나의 진입점으로 집약하는 패키지 초기화 파일이다. 하위 모듈 4개(`validator`, `topology_analyzer`, `integrity_checker`, `catalog_validator`)에서 주요 심볼을 임포트한 후 `__all__`로 재노출한다. 외부에서는 이 패키지 경로만 임포트하면 검증 관련 모든 공개 API에 접근할 수 있다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈
- [`./validator.py`](validator.py.md) — `A2uiValidator`, `A2uiValidatorError`, `ValidationConfig`
- [`./topology_analyzer.py`](topology_analyzer.py.md) — `analyze_topology`
- [`./integrity_checker.py`](integrity_checker.py.md) — `validate_component_integrity`, `validate_recursion_and_paths`, `get_component_references`
- [`./catalog_validator.py`](catalog_validator.py.md) — `CatalogValidator`, `ModelCatalogValidator`, `JsonCatalogValidator`

## Exports

`__all__`에 선언된 공개 심볼 목록:

| 이름 | 원본 모듈 | 종류 |
|---|---|---|
| `A2uiValidator` | `validator` | 클래스 |
| `A2uiValidatorError` | `validator` | 예외 클래스 |
| `ValidationConfig` | `validator` | 클래스 (Pydantic 모델) |
| `analyze_topology` | `topology_analyzer` | 함수 |
| `validate_component_integrity` | `integrity_checker` | 함수 |
| `validate_recursion_and_paths` | `integrity_checker` | 함수 |
| `get_component_references` | `integrity_checker` | 함수 |
| `CatalogValidator` | `catalog_validator` | 클래스 |
| `ModelCatalogValidator` | `catalog_validator` | 클래스 |
| `JsonCatalogValidator` | `catalog_validator` | 클래스 |

## 동작 흐름

모듈 로딩 시 4개 하위 모듈에서 심볼을 임포트하고 `__all__`에 등록하는 것 외에 실행 로직은 없다. 이 파일을 통해 패키지 사용자는 `from a2ui.core.validating import A2uiValidator` 형태로 개별 심볼에 접근하거나 `from a2ui.core.validating import *` 형태로 모든 공개 심볼을 한꺼번에 가져올 수 있다.
