# agent_sdks/python/a2ui_agent/src/a2ui/adk/send_a2ui_to_client_toolset.py

## 개요

ADK 에이전트가 A2UI JSON 페이로드를 클라이언트에 전송할 수 있도록 하는 툴셋 모듈이다. `SendA2uiToClientToolset`은 A2UI 활성화 여부, 카탈로그, 예제를 동적으로 제공받아 LLM이 `send_a2ui_json_to_client` 도구를 호출할 수 있게 한다. LLM 요청 시 시스템 프롬프트에 A2UI 스키마와 예제를 자동 주입하고, 도구 호출 시 JSON을 파싱·검증해 클라이언트로 전달한다. `@experimental` 데코레이터가 적용되어 있다.

## 의존성

### 외부 패키지
- `inspect`, `logging`
- `typing` — `Any`, `Awaitable`, `Callable`, `Optional`, `TYPE_CHECKING`, `TypeAlias`, `Union`
- `google.adk.agents.readonly_context.ReadonlyContext`
- `google.adk.models.llm_request.LlmRequest`
- `google.adk.tools.base_tool.BaseTool`
- `google.adk.tools.base_toolset.BaseToolset`
- `google.adk.tools.tool_context.ToolContext`
- `google.adk.utils.feature_decorator.experimental`
- `google.genai.types as genai_types`

### 저장소 내부 모듈
- [`a2ui.adk.a2a.event_converter`](./a2a/event_converter.py.md) — `A2uiEventConverter`
- [`a2ui.adk.a2a.part_converter`](./a2a/part_converter.py.md) — `A2uiPartConverter`
- `a2ui.parser.payload_fixer` — `parse_and_fix`
- `a2ui.schema.catalog` — `A2uiCatalog`, `CatalogConfig`
- `a2ui.schema.constants` — `A2UI_SCHEMA_BLOCK_END`, `A2UI_SCHEMA_BLOCK_START`, `A2UI_TOOL_ERROR_KEY`, `A2UI_TOOL_NAME`, `A2UI_VALIDATED_JSON_KEY`

### TYPE_CHECKING 전용
- `google.adk.agents.invocation_context.InvocationContext`
- `google.adk.events.event.Event`

## Exports

- `A2uiEnabledProvider` (TypeAlias)
- `A2uiCatalogProvider` (TypeAlias)
- `A2uiExamplesProvider` (TypeAlias)
- `SendA2uiToClientToolset` (클래스, `@experimental`)

## 상세 명세

### 타입 별칭

- `A2uiEnabledProvider`: `Callable[[ReadonlyContext], Union[bool, Awaitable[bool]]]` — A2UI 활성화 여부를 반환하는 동기/비동기 호출 가능 객체 타입.
- `A2uiCatalogProvider`: `Callable[[ReadonlyContext], Union[catalog.A2uiCatalog, Awaitable[catalog.A2uiCatalog]]]` — A2UI 카탈로그를 반환하는 동기/비동기 호출 가능 객체 타입.
- `A2uiExamplesProvider`: `Callable[[ReadonlyContext], Union[str, Awaitable[str]]]` — A2UI 예제 문자열을 반환하는 동기/비동기 호출 가능 객체 타입.

### `SendA2uiToClientToolset(BaseToolset)`

`@experimental` 데코레이터가 적용된 메인 툴셋 클래스.

#### `__init__(self, a2ui_enabled: Union[bool, A2uiEnabledProvider], a2ui_catalog: Union[catalog.A2uiCatalog, A2uiCatalogProvider], a2ui_examples: Union[str, A2uiExamplesProvider])`

- `self._a2ui_enabled`: A2UI 활성화 여부. bool 또는 provider callable.
- `self._ui_tools`: `[_SendA2uiJsonToClientTool(a2ui_catalog, a2ui_examples)]` 리스트로 초기화.

#### `async _resolve_a2ui_enabled(self, ctx: ReadonlyContext) -> bool`

`self._a2ui_enabled`가 `bool`이면 그대로 반환한다. callable이면 호출하고, 결과가 awaitable이면 `await`한다.

#### `async get_tools(self, readonly_context: Optional[ReadonlyContext] = None) -> list[BaseTool]`

A2UI가 활성화되어 있으면 `self._ui_tools`를, 아니면 빈 리스트를 반환한다. `readonly_context`가 `None`이면 `use_ui = False`로 처리한다.

#### `async get_part_converter(self, ctx: ReadonlyContext) -> A2uiPartConverter`

`self._ui_tools[0]._resolve_a2ui_catalog(ctx)`로 카탈로그를 해석하고, `A2uiPartConverter(catalog)`를 생성해 반환한다.

### `_SendA2uiJsonToClientTool(BaseTool)` (비공개 내부 클래스)

LLM에 노출되는 실제 도구 클래스.

#### 클래스 상수
- `TOOL_NAME = A2UI_TOOL_NAME` — 도구 이름
- `VALIDATED_A2UI_JSON_KEY = A2UI_VALIDATED_JSON_KEY` — 검증된 JSON의 반환 딕셔너리 키
- `A2UI_JSON_ARG_NAME = "a2ui_json"` — LLM이 전달하는 인자 이름
- `TOOL_ERROR_KEY = A2UI_TOOL_ERROR_KEY` — 에러 반환 딕셔너리 키

#### `__init__(self, a2ui_catalog: Union[catalog.A2uiCatalog, A2uiCatalogProvider], a2ui_examples: Union[str, A2uiExamplesProvider])`

`BaseTool.__init__`에 `name=TOOL_NAME`, `description`(도구 역할 설명 + 스키마 블록 경계 상수 언급 포함)을 전달한다.

#### `_get_declaration(self) -> genai_types.FunctionDeclaration | None`

GenAI에 등록할 함수 선언을 반환한다. 파라미터로 `a2ui_json` (타입: `STRING`, 필수) 하나를 가진 `OBJECT` 스키마를 정의한다.

#### `async _resolve_a2ui_examples(self, ctx: ReadonlyContext) -> str`

`self._a2ui_examples`가 `str`이면 그대로 반환한다. callable이면 호출하고 awaitable이면 `await`한다.

#### `async _resolve_a2ui_catalog(self, ctx: ReadonlyContext) -> catalog.A2uiCatalog`

`self._a2ui_catalog`가 `catalog.A2uiCatalog` 인스턴스이면 그대로 반환한다. callable이면 호출하고 awaitable이면 `await`한다.

#### `async process_llm_request(self, *, tool_context: ToolContext, llm_request: LlmRequest) -> None`

LLM 요청 전처리 훅. 부모 클래스 메서드 호출 후:
1. `_resolve_a2ui_catalog`로 카탈로그를 해석한다.
2. `a2ui_catalog.render_as_llm_instructions()`로 카탈로그를 LLM 지시문으로 렌더링한다.
3. `_resolve_a2ui_examples`로 예제 문자열을 해석한다.
4. `llm_request.append_instructions([instruction, examples])`로 시스템 지시문에 추가한다.

#### `async run_async(self, *, args: dict[str, Any], tool_context: ToolContext) -> Any`

도구 실행 메서드. 예외를 전체 try-except로 감싼다.

1. `args.get("a2ui_json")`로 JSON 문자열을 가져온다. 없으면 `ValueError` 발생.
2. `_resolve_a2ui_catalog`로 카탈로그를 해석한다.
3. `parse_and_fix(a2ui_json)`으로 JSON 문자열을 파싱 및 수정한다.
4. `a2ui_catalog.validator.validate(a2ui_json_payload)`로 스키마 검증을 수행한다.
5. `tool_context.actions.skip_summarization = True`로 설정해 LLM 요약 추론 호출을 방지한다.
6. `{VALIDATED_A2UI_JSON_KEY: a2ui_json_payload}`를 반환한다.
7. 예외 발생 시 에러 로그를 기록하고 `{TOOL_ERROR_KEY: err}`를 반환한다.

## 동작 흐름

에이전트 초기화 시 `SendA2uiToClientToolset`을 tools 목록에 추가한다. 각 LLM 요청 전 `process_llm_request`가 A2UI 스키마와 예제를 시스템 프롬프트에 주입한다. LLM이 `send_a2ui_json_to_client` 도구를 호출하면 `run_async`가 JSON을 검증하고 결과를 반환한다. 이후 `A2uiPartConverter`(또는 `A2uiEventConverter`)가 이 결과를 A2A `Part`로 변환한다.
