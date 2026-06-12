# agent_sdks/python/a2ui_agent/src/a2ui/adk/a2a/part_converter.py

## 개요

GenAI(Google Generative AI) 파트를 A2A 파트로 변환하는 카탈로그 인식 컨버터 모듈이다. 도구 호출 응답(`FunctionResponse`)과 텍스트 파트 모두를 처리하며, A2UI 전용 도구(`send_a2ui_json_to_client`) 응답에서 검증된 JSON을 추출하거나, 텍스트/도구 응답에서 A2UI 딜리미터 태그가 포함된 경우 이를 파싱해 A2UI `Part`로 변환한다. `@experimental` 데코레이터가 적용되어 있다.

## 의존성

### 외부 패키지
- `logging`
- `typing` — `Optional`
- `a2a.types as a2a_types`
- `google.adk.a2a.converters.part_converter` — `convert_genai_part_to_a2a_part` (기본 변환 함수)
- `google.adk.utils.feature_decorator.experimental`
- `google.genai.types as genai_types`

### 저장소 내부 모듈
- [`a2ui.a2a.parts`](../../a2a/parts.py.md) — `create_a2ui_part`, `parse_response_to_parts`
- `a2ui.parser.parser` — `has_a2ui_parts`
- `a2ui.schema.constants` — `A2UI_TOOL_NAME`, `A2UI_TOOL_ERROR_KEY`, `A2UI_VALIDATED_JSON_KEY`, `VERSION_0_8`
- `a2ui.schema.catalog` — `A2uiCatalog`

## Exports

- `A2uiPartConverter` (클래스, `@experimental`)

## 상세 명세

### `A2uiPartConverter`

`@experimental` 데코레이터가 적용된 카탈로그 인식 GenAI → A2A 파트 컨버터 클래스.

#### `__init__(self, a2ui_catalog: A2uiCatalog, bypass_tool_check: bool = False, fallback_text: Optional[str] = None, version: str = constants.VERSION_0_8)`

- `self._catalog`: 검증에 사용할 `A2uiCatalog` 인스턴스.
- `self._bypass_tool_check`: `True`이면 도구 이름이 `A2UI_TOOL_NAME`과 일치하지 않아도 FunctionResponse를 A2UI로 처리한다. 기본값 `False`.
- `self._fallback_text`: 파싱 실패 시 대체 텍스트. 기본값 `None`.
- `self._version`: 생성할 A2UI 파트의 버전. 기본값 `constants.VERSION_0_8`(값은 `"0.8"`).

#### `convert(self, part: genai_types.Part) -> list[a2a_types.Part]`

단일 GenAI 파트를 A2A 파트 리스트로 변환한다. 네 가지 경우를 순서대로 처리한다.

**1. FunctionResponse 처리 (도구 응답)**

- `part.function_response`가 있을 경우, `function_response.name`이 `constants.A2UI_TOOL_NAME`인지 또는 `self._bypass_tool_check`가 `True`인지 확인한다.
- 위 조건이 충족되면:
  - `response_dict`에 `A2UI_TOOL_ERROR_KEY`가 있으면 경고 로그 출력 후 빈 리스트를 반환한다.
  - `response_dict`가 딕셔너리이고 `A2UI_VALIDATED_JSON_KEY`를 포함하면, 해당 JSON 데이터로 `create_a2ui_part` 리스트를 생성해 반환한다.
  - `is_send_a2ui_json_to_client_response`가 `True`이면 INFO 로그 후 빈 리스트를 반환한다.
- 위 조건과 관계없이(`bypass_tool_check=False`인 다른 도구), `function_response.response`의 `"result"` 키에 문자열이 있고 `has_a2ui_parts(result)`가 True이면, `parse_response_to_parts`로 파싱해 반환한다.

**2. FunctionCall 처리 (도구 호출)**

- `part.function_call`이 있고 그 `name`이 `constants.A2UI_TOOL_NAME`이면, 클라이언트에 직접 전달하지 않기 위해 빈 리스트를 반환한다.

**3. 텍스트 기반 A2UI 처리**

- `part.text`가 있고 `has_a2ui_parts(text)`가 True이면, `parse_response_to_parts`로 파싱해 반환한다.

**4. 기본 변환 (폴백)**

- 위 세 경우에 해당하지 않으면, `part_converter.convert_genai_part_to_a2a_part(part)`로 기본 변환 후 결과가 있으면 단일 요소 리스트로, 없으면 빈 리스트로 반환한다.

## 동작 흐름

`A2uiEventConverter`에 의해 인스턴스화되어 각 GenAI 파트를 처리한다. A2UI 전용 도구 응답을 최우선으로 처리하고, 텍스트 스트림에서 A2UI 태그를 감지하는 것을 그 다음으로 처리하며, 일반 파트는 ADK 기본 컨버터에 위임한다. 검증은 `a2ui_catalog.validator`를 통해 `parse_response_to_parts` 내부에서 수행된다.
