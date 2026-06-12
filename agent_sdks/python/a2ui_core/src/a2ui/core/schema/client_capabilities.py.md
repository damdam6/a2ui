# agent_sdks/python/a2ui_core/src/a2ui/core/schema/client_capabilities.py

## 개요

A2UI 클라이언트가 서버에 자신의 능력(capabilities)을 알리기 위해 사용하는 Pydantic 모델들을 정의한다. 자동 생성된 파일이다. 클라이언트가 지원하는 카탈로그 ID 목록, 인라인 카탈로그 정의, 각 함수 정의를 구조화된 방식으로 표현한다.

## 의존성

### 외부 패키지
- `typing` — `Any`, `Dict`, `List`, `Literal`, `Optional`
- `pydantic` — `BaseModel`, `Field`, `ConfigDict`

### 저장소 내부 모듈
- [`./common_types.py`](common_types.py.md) — `StrictBaseModel`
- [`./constants.py`](constants.py.md) — `SPEC_VERSION`

## Exports

| 이름 | 종류 |
|---|---|
| `FunctionDefinition` | 클래스 (Pydantic 모델) |
| `InlineCatalog` | 클래스 (Pydantic 모델) |
| `V09Capabilities` | 클래스 (Pydantic 모델) |
| `A2uiClientCapabilities` | 클래스 (Pydantic 모델) |

## 상세 명세

### `FunctionDefinition(StrictBaseModel)`

카탈로그 내 개별 함수 하나의 정의를 표현한다.

| 필드명 | 타입 | alias | 필수 여부 | 설명 |
|---|---|---|---|---|
| `name` | `str` | — | 필수 | 함수의 고유 이름. |
| `description` | `Optional[str]` | — | 선택 (`None`) | 함수의 용도와 사용법에 대한 설명. |
| `parameters` | `Any` | — | 필수 | 함수 인수(`args`)를 기술하는 JSON Schema. |
| `return_type` | `Literal["string", "number", "boolean", "array", "object", "any", "void"]` | `"returnType"` | 필수 | 함수 반환 타입. |

`StrictBaseModel` 상속으로 extra 필드 금지, `populate_by_name=True` 적용.

---

### `InlineCatalog(StrictBaseModel)`

클라이언트가 인라인으로 제공하는 카탈로그 전체 정의를 표현한다. 에이전트가 `acceptsInlineCatalogs: true`를 선언한 경우에만 사용된다.

| 필드명 | 타입 | alias | 필수 여부 | 설명 |
|---|---|---|---|---|
| `catalog_id` | `str` | `"catalogId"` | 필수 | 카탈로그의 고유 식별자. |
| `components` | `Optional[Dict[str, Any]]` | — | 선택 | UI 컴포넌트 정의 맵. |
| `functions` | `Optional[List[FunctionDefinition]]` | — | 선택 | 지원하는 함수 정의 목록. |
| `theme` | `Optional[Dict[str, Any]]` | — | 선택 | 테마 속성 스키마. 키가 테마 속성명, 값이 해당 속성의 JSON Schema. |

---

### `V09Capabilities(StrictBaseModel)`

v0.9 명세에 대한 클라이언트 능력을 표현한다.

| 필드명 | 타입 | alias | 필수 여부 | 설명 |
|---|---|---|---|---|
| `supported_catalog_ids` | `List[str]` | `"supportedCatalogIds"` | 필수 | 클라이언트가 지원하는 카탈로그 URI 목록. |
| `inline_catalogs` | `Optional[List[InlineCatalog]]` | `"inlineCatalogs"` | 선택 | 인라인 카탈로그 정의 배열. |

---

### `A2uiClientCapabilities(StrictBaseModel)`

클라이언트 능력의 최상위 래퍼 모델. 버전별 능력을 키로 보관한다.

| 필드명 | 타입 | alias | 필수 여부 | 설명 |
|---|---|---|---|---|
| `v0_9` | `Optional[V09Capabilities]` | `SPEC_VERSION` (`"v0.9"`) | 선택 | v0.9 버전 능력 선언. |

`SPEC_VERSION` 상수를 alias로 사용하므로, JSON에서의 키 이름은 `"v0.9"`이다.

## 동작 흐름

이 파일은 데이터 컨테이너 역할만 한다. 클라이언트는 `A2uiClientCapabilities` 인스턴스를 JSON으로 직렬화해 서버에 전송한다. `populate_by_name=True` 덕분에 Python 필드명(`v0_9`)과 JSON 키(`v0.9`) 모두로 값을 채울 수 있다.
