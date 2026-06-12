# agent_sdks/python/a2ui_core/src/a2ui/core/basic_catalog/styles.py

## 개요

A2UI 서피스(surface)의 시각적 스타일 테마를 정의하는 Pydantic 모델 `Theme`을 포함한다. 자동 생성된 파일로 직접 편집하지 않는다. 주된 역할은 에이전트 또는 도구와 연관된 브랜드 색상·아이콘 URL·표시 이름을 구조화된 방식으로 표현하는 것이다.

## 의존성

### 외부 패키지
- `typing` — `Any`, `Dict`, `List`, `Literal`, `Optional`, `Union`
- `pydantic` — `BaseModel`, `Field`, `ConfigDict`

### 저장소 내부 모듈
- [`../schema/common_types.py`](../schema/common_types.py.md) — `StrictBaseModel` (import되지만 현재 파일에서 직접 사용되지 않음)

## Exports

| 이름 | 종류 |
|---|---|
| `Theme` | 클래스 (Pydantic 모델) |

## 상세 명세

### `Theme(BaseModel)`

`BaseModel`을 직접 상속한다(`StrictBaseModel`이 아님). 따라서 extra 필드 금지 등의 엄격한 검증이 적용되지 않는다.

**model_config**: `ConfigDict(populate_by_name=True)` — `alias`로도, 원래 필드명으로도 값을 채울 수 있다.

#### 필드

| 필드명 | 타입 | alias | 기본값 | 설명 |
|---|---|---|---|---|
| `primary_color` | `Optional[str]` | `"primaryColor"` | `None` | 주요 브랜드 색상. 버튼 강조, 활성 테두리 등에 사용. 형식 검증 정규식: `^#[0-9a-fA-F]{6}$` (6자리 16진수 hex). 렌더러가 문맥에 따라 파생 색상을 생성할 수 있다. |
| `icon_url` | `Optional[str]` | `"iconUrl"` | `None` | 서피스와 연결된 에이전트 또는 도구를 식별하는 이미지 URL. |
| `agent_display_name` | `Optional[str]` | `"agentDisplayName"` | `None` | 서피스 옆에 표시되어 에이전트/도구를 식별하는 텍스트. |

모든 필드가 `Optional`이므로 `Theme()`처럼 아무 인수 없이 생성 가능하다.

## 동작 흐름

파일은 모델 선언만 포함한다. `Theme` 인스턴스는 카탈로그 정의 시 `theme` 필드에 사용되고, `ModelCatalog.get_theme_schema()`가 `Theme.model_json_schema()`를 호출해 JSON Schema를 추출한다. `primary_color`는 Pydantic의 `pattern` 검증이 적용되어 잘못된 hex 형식이 입력되면 `ValidationError`가 발생한다.
