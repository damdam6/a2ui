# agent_sdks/python/a2ui_core/tests/test_components.py

## 개요

`BasicCatalog`에 포함된 핵심 컴포넌트 모델(`ImageComponent`, `TextComponent`, `ButtonComponent`, `Theme`, `AnyComponent`)과 메시지 스키마 클래스(`UpdateComponentsMessage`, `A2uiMessageListWrapper`)의 pydantic 유효성 검사 동작을 검증하는 pytest 테스트 파일이다. 판별 유니온(`AnyComponent`) 라우팅, 엄격한 추가 속성 금지(`extra="forbid"`), 메시지 페이로드 파싱, JSON 스키마 생성, `Theme`의 추가 속성 허용 동작 등을 망라하여 검증한다.

## 의존성

### 외부 패키지
- `pytest`
- `pydantic` — `ValidationError`, `TypeAdapter`

### 저장소 내부 모듈
- [`agent_sdks/python/a2ui_core/src/a2ui/core/basic_catalog/__init__.py`](../src/a2ui/core/basic_catalog/__init__.py.md) — `ImageComponent`, `TextComponent`, `ButtonComponent`, `Theme`, `AnyComponent`
- [`agent_sdks/python/a2ui_core/src/a2ui/core/schema/__init__.py`](../src/a2ui/core/schema/__init__.py.md) — `UpdateComponentsMessage`, `A2uiMessageListWrapper`

## Exports

테스트 함수만 정의하며 외부로 export되는 공개 심벌은 없다.

## 상세 명세 (테스트 케이스)

### `test_image_component_valid_with_description()`
**검증 동작:** `ImageComponent.model_validate()`로 `url`과 `description`이 모두 있는 payload를 파싱했을 때 `component == "Image"`, `url`, `description` 필드가 정확히 매핑되는지 확인한다.
- payload: `{"id": "img-1", "component": "Image", "url": "https://example.com/image.png", "description": "An example image"}`
- **픽스처/모킹:** 없음.

### `test_image_component_valid_without_description()`
**검증 동작:** `description` 필드 없이 생성했을 때 `parsed.description is None`임을 확인하여 `description`이 Optional임을 검증한다.
- **픽스처/모킹:** 없음.

### `test_image_component_invalid_field_type()`
**검증 동작:** `url` 필드에 정수(`123`)를 전달하면 `ValidationError`가 발생해야 한다. `url`은 string 또는 DataBinding이어야 하므로 정수는 허용되지 않는다.
- **픽스처/모킹:** 없음.

### `test_any_component_discriminated_union()`
**검증 동작:** `TypeAdapter(AnyComponent)`가 `component` 필드 값에 따라 올바른 구체 타입으로 라우팅하는지 세 가지 경우를 검증한다:
1. `"component": "Text"` → 결과가 `TextComponent` 인스턴스이고 `text == "Hello, A2UI!"`, `variant == "h1"`.
2. `"component": "Image"` → 결과가 `ImageComponent` 인스턴스이고 `url == "https://example.com/photo.jpg"`.
3. `"component": "Button"` → 결과가 `ButtonComponent` 인스턴스이고 `child == "text-1"`, `action.event.name == "click"`. Button payload에는 `action: {"event": {"name": "click", "context": {}}}` 포함.
- **픽스처/모킹:** 없음.

### `test_any_component_invalid_discriminator()`
**검증 동작:** `component: "UnknownComponentType"`처럼 등록되지 않은 판별자 값이면 `AnyComponent` 유니온 유효성 검사에서 `ValidationError`가 발생해야 한다.
- **픽스처/모킹:** 없음.

### `test_text_component_validation()`
**검증 동작:**
1. `TextComponent(id="welcome_text", component="Text", text="Hello World!", variant="h1")`으로 직접 생성하면 각 필드가 올바르게 설정된다.
2. `TextComponent(id="bad")`처럼 필수 필드(`component`, `text`)가 없으면 `ValidationError`가 발생한다.
- **픽스처/모킹:** 없음.

### `test_text_component_variant_enum()`
**검증 동작:**
1. `variant`를 생략한 `TextComponent`는 기본값 `"body"`가 적용되어야 한다.
2. 허용되지 않는 `variant` 값(`"bold"`)을 전달하면 `ValidationError`가 발생하며, 오류 메시지에 `"Input should be 'h1', 'h2', 'h3', 'h4', 'h5', 'caption' or 'body'"` 문자열이 포함되어야 한다.
- **픽스처/모킹:** 없음.

### `test_button_component_strict_extra_forbid()`
**검증 동작:** `ButtonComponent`(StrictBaseModel 기반, `extra="forbid"`)에 `extra_invalid_field="not allowed"`를 전달하면 `ValidationError`가 발생해야 한다.
- payload: `{"id": "btn", "component": "Button", "child": "welcome_text", "action": {"event": {"name": "click"}}, "extra_invalid_field": "not allowed"}`
- **픽스처/모킹:** 없음.

### `test_message_payload_parsing()`
**검증 동작:** `UpdateComponentsMessage.model_validate()`로 다음 구조를 파싱한다:
- 최상위: `{"version": "v0.9", "updateComponents": {...}}`
- 내부 `updateComponents`: `{"surfaceId": "surface_1", "components": [Text dict, Button dict]}`

파싱 후 `msg.version == "v0.9"`, `msg.update_components.surface_id == "surface_1"`, 컴포넌트 목록 길이가 2이고 각 dict의 `"component"` 값이 `"Text"`, `"Button"`인지 확인한다. 중요: `components` 필드는 파싱된 모델 인스턴스가 아닌 dict 목록으로 저장된다.
- **픽스처/모킹:** 없음.

### `test_model_json_schema_generation()`
**검증 동작:** `A2uiMessageListWrapper.model_json_schema()`가 None이 아닌 dict를 반환하며, `"properties"` 키가 존재하고 그 안에 `"messages"` 키가 포함되어야 한다.
- **픽스처/모킹:** 없음.

### `test_theme_allows_additional_properties()`
**검증 동작:** `Theme` 모델의 추가 속성 허용 여부를 세 단계로 검증한다:
1. `Theme(primary_color="#00BFFF")` — 정상 생성, `primary_color == "#00BFFF"`.
2. `Theme(primary_color="#00BFFF", extra_custom_color="#FF0000")` — 미정의 속성 포함도 `ValidationError` 없이 생성되어야 한다. 이는 catalog.json에서 `additionalProperties: True`를 사용하므로 `Theme`이 동적으로 추가 속성을 허용하도록 구성되었기 때문이다.
3. 대조군으로, `ButtonComponent`에 `extra_garbage_property="forbidden"` 전달 시 `ValidationError`가 발생함을 확인한다.
- **픽스처/모킹:** 없음.

### `test_text_component_discriminator_behavior()`
**검증 동작:** `component` 판별자 필드의 두 가지 동작 경로를 테스트한다:
1. Python 직접 인스턴스화 — `TextComponent(id="text_1", text="Direct Instantiation works!")`처럼 `component`를 명시하지 않아도, pydantic이 `Literal["Text"]` 기본값을 적용하여 `comp.component == "Text"`가 된다.
2. JSON 유효성 검사:
   - `"component": "Text"`가 있는 payload — `TypeAdapter(AnyComponent).validate_python()`으로 `TextComponent` 인스턴스 반환.
   - `"component"` 키가 없는 payload(`{"id": "text_3", "text": "Missing component key"}`) — `ValidationError`가 발생하며, 오류 메시지에 `"Input tag 'component' is missing"` 또는 `"union_tag_not_found"`가 포함되어야 한다.
- **픽스처/모킹:** 없음.

## 동작 흐름

모든 테스트는 독립적이며 공유 픽스처가 없다. 각 테스트는 payload dict를 직접 구성하여 pydantic `model_validate()`, `TypeAdapter.validate_python()`, 또는 직접 생성자 호출로 모델을 생성한 뒤, 필드 값 단언(`assert`) 또는 `pytest.raises()`로 예외 발생을 확인한다. 테스트 범위는 개별 컴포넌트 모델 파싱에서 시작하여 판별 유니온 라우팅, 메시지 래퍼 파싱, JSON 스키마 생성 순으로 계층적으로 확대된다.
