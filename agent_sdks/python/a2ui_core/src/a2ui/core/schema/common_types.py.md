# agent_sdks/python/a2ui_core/src/a2ui/core/schema/common_types.py

## 개요

A2UI 스키마 시스템 전반에서 공유되는 기반 타입과 구조체를 정의한다. 자동 생성된 파일로 직접 편집하지 않는다. 컴포넌트 참조 마커 클래스, 엄격한 Pydantic 기반 모델, 동적 데이터 바인딩과 함수 호출 타입, 컴포넌트 공통 구조 등이 모두 이 파일에서 출발한다.

## 의존성

### 외부 패키지
- `typing` — `Annotated`, `Any`, `Dict`, `List`, `Literal`, `Optional`, `Union`
- `pydantic` — `BaseModel`, `Field`, `ConfigDict`
- `pydantic_core` — `core_schema` (SingleReference 내부에서 지연 import)

### 저장소 내부 모듈
없음.

## Exports

| 이름 | 종류 |
|---|---|
| `ComponentReference` | 클래스 (마커) |
| `SingleReference` | 클래스 (`str` + `ComponentReference` 다중 상속) |
| `ListReference` | 클래스 (마커) |
| `StrictBaseModel` | 클래스 (Pydantic 모델 기반) |
| `ComponentId` | 타입 별칭 (`SingleReference`) |
| `DataBinding` | 클래스 (Pydantic 모델) |
| `FunctionCall` | 클래스 (Pydantic 모델) |
| `DynamicValue` | 타입 별칭 (Union) |
| `DynamicString` | 타입 별칭 (Union) |
| `DynamicNumber` | 타입 별칭 (Union) |
| `DynamicBoolean` | 타입 별칭 (Union) |
| `DynamicStringList` | 타입 별칭 (Union) |
| `TemplateChildList` | 클래스 (Pydantic 모델 + ListReference) |
| `ChildList` | 타입 별칭 (Union) |
| `AccessibilityAttributes` | 클래스 (Pydantic 모델) |
| `CheckRule` | 클래스 (Pydantic 모델) |
| `ActionEvent` | 클래스 (Pydantic 모델) |
| `ActionEventWrapper` | 클래스 (Pydantic 모델) |
| `ActionFunctionCallWrapper` | 클래스 (Pydantic 모델) |
| `Action` | 타입 별칭 (Union) |
| `ComponentCommon` | 클래스 (Pydantic 모델) |

## 상세 명세

### `ComponentReference`
다른 컴포넌트를 참조하는 모든 타입의 기반 마커 클래스. 상속 구조를 통한 타입 식별에만 사용된다.

### `SingleReference(str, ComponentReference)`
단일 컴포넌트 참조를 나타내는 `str` 서브클래스. Pydantic과 통합하기 위해 `__get_pydantic_core_schema__` 클래스 메서드를 구현한다:
- `pydantic_core.core_schema.str_schema()`를 기반으로 하고, 검증 후 자신(`cls`)으로 변환하는 `no_info_after_validator_function`을 사용한다.
- 직렬화 시 `str`으로 변환하는 `plain_serializer_function_ser_schema(str)`를 적용한다.
- 결과적으로 JSON에서는 문자열, Python에서는 `SingleReference` 인스턴스로 동작한다.

### `ListReference(ComponentReference)`
여러 컴포넌트 참조 목록을 보유하는 필드를 표시하는 마커 클래스. 직접 인스턴스화하지 않는다.

### `StrictBaseModel(BaseModel)`
`ConfigDict(extra="forbid", populate_by_name=True)`를 적용한 엄격한 기반 모델. extra 필드를 허용하지 않아 스키마 위반 시 `ValidationError` 발생. Python 필드명과 JSON alias 모두로 값을 채울 수 있다.

### `ComponentId`
`SingleReference`의 별칭. 컴포넌트의 `id` 필드 타입으로 사용.

### `DataBinding(StrictBaseModel)`
데이터 모델 내 값으로의 참조 바인딩.

| 필드명 | 타입 | 필수 | 설명 |
|---|---|---|---|
| `path` | `str` | O | 데이터 모델 내 값을 가리키는 JSON 포인터 경로. |

### `FunctionCall(StrictBaseModel)`
함수 호출을 표현하는 모델.

| 필드명 | 타입 | alias | 필수 | 기본값 | 설명 |
|---|---|---|---|---|---|
| `call` | `str` | — | O | — | 호출할 함수 이름. |
| `args` | `Optional[Dict[str, Any]]` | — | 선택 | `None` | 함수에 전달할 인수. |
| `return_type` | `Optional[Literal["string","number","boolean","array","object","any","void"]]` | `"returnType"` | 선택 | `"boolean"` | 예상 반환 타입. |

### 동적 타입 별칭

- `DynamicValue`: `Union[str, float, bool, List[Any], DataBinding, FunctionCall]` — 임의 동적 값.
- `DynamicString`: `Union[str, DataBinding, FunctionCall]` — 문자열 또는 동적 바인딩.
- `DynamicNumber`: `Union[float, DataBinding, FunctionCall]` — 숫자 또는 동적 바인딩.
- `DynamicBoolean`: `Union[bool, DataBinding, FunctionCall]` — 불리언 또는 동적 바인딩.
- `DynamicStringList`: `Union[List[str], DataBinding, FunctionCall]` — 문자열 목록 또는 동적 바인딩.

### `TemplateChildList(StrictBaseModel, ListReference)`
템플릿 기반 자식 컴포넌트 목록의 동적 참조를 표현한다. `ListReference` 마커도 함께 상속.

| 필드명 | 타입 | alias | 필수 | 설명 |
|---|---|---|---|---|
| `component_id` | `ComponentId` | `"componentId"` | O | 반복할 컴포넌트의 ID. |
| `path` | `str` | — | O | 데이터 모델에서 컴포넌트 속성 객체 목록을 가리키는 경로. |

### `ChildList`
`Union[List[ComponentId], TemplateChildList]` — 정적 자식 목록 또는 템플릿 기반 동적 자식 목록.

### `AccessibilityAttributes(StrictBaseModel)`
보조 기술을 위한 접근성 속성.

| 필드명 | 타입 | 설명 |
|---|---|---|
| `label` | `Optional[DynamicString]` | 1~3 단어의 간결한 레이블 (예: `"Submit"`). |
| `description` | `Optional[DynamicString]` | 추가 설명 (예: 형식 요구 사항, 동작 결과). |

### `CheckRule(StrictBaseModel)`
조건부 검증 규칙.

| 필드명 | 타입 | 설명 |
|---|---|---|
| `condition` | `DynamicBoolean` | 검사할 조건. |
| `message` | `str` | 조건 불충족 시 표시할 오류 메시지. |

### `ActionEvent(StrictBaseModel)`
서버로 전달할 액션 이벤트 명세.

| 필드명 | 타입 | 설명 |
|---|---|---|
| `name` | `str` | 서버에 전달할 액션 이름. |
| `context` | `Optional[Dict[str, Any]]` | 액션 컨텍스트 키-값 쌍. 정적 값은 리터럴, 동적 값은 경로를 사용한다. 정적 ID에는 경로를 사용하지 않는다. |

### `ActionEventWrapper(StrictBaseModel)`
`event: ActionEvent = Field(...)` — `ActionEvent`를 담는 단순 래퍼.

### `ActionFunctionCallWrapper(StrictBaseModel)`
`function_call: FunctionCall = Field(..., alias="functionCall")` — `FunctionCall`을 담는 단순 래퍼.

### `Action`
`Union[ActionEventWrapper, ActionFunctionCallWrapper]` — 이벤트 디스패치 또는 로컬 함수 호출 중 하나를 취하는 액션.

### `ComponentCommon(StrictBaseModel)`
모든 컴포넌트가 공통으로 갖는 기반 필드.

| 필드명 | 타입 | 설명 |
|---|---|---|
| `id` | `ComponentId` | 컴포넌트의 고유 식별자. |
| `accessibility` | `Optional[AccessibilityAttributes]` | 접근성 속성 (선택). |

## 동작 흐름

이 파일은 순수 타입 정의만 포함한다. `SingleReference`의 Pydantic 통합은 import 시점에 `__get_pydantic_core_schema__`가 Pydantic 코어 스키마를 반환하는 방식으로 이루어지며, 이를 통해 JSON 문자열 → `SingleReference` 인스턴스 변환이 자동화된다. 동적 타입 별칭들은 A2UI의 핵심 개념인 "값이 리터럴일 수도, 데이터 바인딩일 수도, 함수 호출일 수도 있다"를 타입 시스템에서 표현한다.
