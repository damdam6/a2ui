# agent_sdks/python/a2ui_agent/tests/integration/verify_load_real.py

## 개요

이 파일은 A2UI Python SDK의 통합 검증 스크립트로, `A2uiSchemaManager`와 `BasicCatalog`가 실제 스키마 파일을 올바르게 로드하고, 실제 메시지 페이로드를 유효성 검사할 수 있는지 확인한다. `VERSION_0_8`과 `VERSION_0_9` 두 버전을 순서대로 검증하며, 어느 하나라도 실패하면 `sys.exit(1)`로 종료한다. pytest가 아닌 단독 실행 스크립트 형태로 작성되어 있다.

## 의존성

### 외부 패키지
- `sys` (표준 라이브러리)

### 저장소 내부 모듈
- [`agent_sdks/python/a2ui_agent/src/a2ui/schema/manager.py`](../../src/a2ui/schema/manager.py.md) — `A2uiSchemaManager`
- [`agent_sdks/python/a2ui_agent/src/a2ui/schema/constants.py`](../../src/a2ui/schema/constants.py.md) — `CATALOG_COMPONENTS_KEY`, `VERSION_0_8`, `VERSION_0_9`
- [`agent_sdks/python/a2ui_agent/src/a2ui/schema/common_modifiers.py`](../../src/a2ui/schema/common_modifiers.py.md) — `remove_strict_validation`
- [`agent_sdks/python/a2ui_agent/src/a2ui/basic_catalog/provider.py`](../../src/a2ui/basic_catalog/provider.py.md) — `BasicCatalog`

## Exports

이 파일은 모듈로서 명시적인 `__all__` 선언이 없다. 최상위에 정의된 항목은 다음과 같다.

| 이름 | 종류 |
|------|------|
| `verify` | 함수 |

## 상세 명세

### `verify() -> None`

**시그니처:** `def verify():`

**동작 로직:**

**1단계 — VERSION_0_8 검증:**

1. `"Verifying A2uiSchemaManager..."` 를 출력한다.
2. `try` 블록 안에서 `A2uiSchemaManager`를 `version=VERSION_0_8`, `catalogs=[BasicCatalog.get_config(VERSION_0_8)]`, `schema_modifiers=[remove_strict_validation]`로 인스턴스화한다.
3. `manager.get_selected_catalog()`를 호출해 카탈로그 객체를 얻는다.
4. `catalog.catalog_schema[CATALOG_COMPONENTS_KEY]`로 컴포넌트 딕셔너리를 꺼낸 뒤, 컴포넌트 수와 앞 5개 키를 출력한다.
5. `surfaceId: "contact-card"`, `root: "main_card"`를 기준으로 구성된 `a2ui_message` 리스트를 인라인으로 정의한다. 이 메시지는 세 개의 항목으로 구성된다:
   - `beginRendering` 메시지: `surfaceId = "contact-card"`, `root = "main_card"`
   - `surfaceUpdate` 메시지: 연락처 카드 UI를 표현하는 30개 이상의 컴포넌트 정의 (`Image`, `Text`, `Column`, `Row`, `Icon`, `Divider`, `Button`, `Card` 등). 각 컴포넌트는 `id`와 `component` 키를 가지며, 일부는 `weight`를 추가로 가진다. 값 참조는 `{"path": "/필드명"}` 또는 `{"literalString": "값"}` 형식을 사용한다.
   - `dataModelUpdate` 메시지: `surfaceId = "contact-card"`, `path = "/"`, `contents`에 `name`, `title`, `team`, `location`, `email`, `mobile`, `calendar`, `imageUrl`, `contacts` 키를 포함한 배열
6. `catalog.validator.validate(a2ui_message)`를 호출한다. 성공 시 `"Validation successful"`을 출력한다.
7. `except Exception as e:` 에서 실패 메시지를 출력하고 `sys.exit(1)`을 호출한다.

**2단계 — VERSION_0_9 검증:**

1. 별도의 `try` 블록에서 동일한 패턴으로 `A2uiSchemaManager`를 `version=VERSION_0_9`로 인스턴스화한다.
2. 카탈로그를 얻고 전체 컴포넌트 키 목록을 출력한다.
3. `a2ui_message`를 v0.9 형식으로 정의한다. 이 메시지는 네 개의 항목으로 구성된다. 각 항목은 최상위에 `"version": "v0.9"` 키를 포함한다:
   - `createSurface`: `surfaceId = "contact_form_1"`, `catalogId = "https://a2ui.dev/specification/v0_9/catalogs/basic/catalog.json"`, `fakeProperty = "should be allowed"` (허용되는 추가 필드 확인용)
   - `updateComponents`: `surfaceId = "contact_form_1"`, `components` 배열에 연락처 폼 UI 컴포넌트들 (`Card`, `Column`, `Row`, `Icon`, `Text`, `TextField`, `ChoicePicker`, `Divider`, `CheckBox`, `Button`). v0.9에서 컴포넌트는 `{"component": "컴포넌트명", "id": "...", ...}` 형식(플랫 구조)을 사용한다. 필드 유효성 검사는 `checks` 배열로 정의하며, `required`, `email`, `regex` 함수를 조건으로 사용한다.
   - `updateDataModel`: `surfaceId = "contact_form_1"`, `path = "/contact"`, `value`에 `firstName`, `lastName`, `email`, `phone`, `preference`, `subscribe` 키를 포함한 딕셔너리
   - `deleteSurface`: `surfaceId = "contact_form_1"`
4. `catalog.validator.validate(a2ui_message)`를 호출하고, 성공/실패를 동일한 패턴으로 처리한다.

**에러 처리:** 각 버전 블록이 독립적인 `try/except`로 감싸져 있어, 첫 번째 버전 실패 시 두 번째 버전 검증은 실행되지 않는다.

## 동작 흐름

```
__main__ 진입
  → verify() 호출
    → [VERSION_0_8 블록]
        A2uiSchemaManager 생성 (BasicCatalog 사용, strict 검증 비활성화)
        → 카탈로그 로드 확인 (컴포넌트 수 출력)
        → 실제 연락처 카드 메시지 구성 (beginRendering + surfaceUpdate + dataModelUpdate)
        → validator.validate() 호출
        → 실패 시 sys.exit(1)
    → [VERSION_0_9 블록]
        A2uiSchemaManager 생성 (v0.9 BasicCatalog 사용)
        → 카탈로그 로드 확인
        → 실제 연락처 폼 메시지 구성 (createSurface + updateComponents + updateDataModel + deleteSurface)
        → validator.validate() 호출
        → 실패 시 sys.exit(1)
```

스크립트는 `if __name__ == '__main__':` 가드를 통해 직접 실행 시에만 `verify()`를 호출한다.
