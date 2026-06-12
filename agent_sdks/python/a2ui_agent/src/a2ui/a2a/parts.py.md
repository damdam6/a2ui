# agent_sdks/python/a2ui_agent/src/a2ui/a2a/parts.py

## 개요

A2A(Agent-to-Agent) 프로토콜의 `Part` 타입과 A2UI JSON 데이터 간의 변환을 담당하는 유틸리티 모듈이다. A2UI 데이터를 담는 `DataPart` 생성, A2UI 파트 여부 판별, LLM 응답 문자열에서 A2UI 블록을 추출해 `Part` 리스트로 변환하는 함수들을 제공한다. 스트리밍 방식의 비동기 파싱도 지원한다.

## 의존성

### 외부 패키지
- `logging`
- `typing` — `Any`, `Optional`, `List`, `AsyncIterable`, `TYPE_CHECKING`
- `a2a.types` — `Part`, `DataPart`, `TextPart`

### 저장소 내부 모듈 (지연 import)
- `a2ui.parser.streaming.A2uiStreamParser` — TYPE_CHECKING용 타입 힌트
- `a2ui.parser.parser.parse_response` — `parse_response_to_parts` 내부에서 지연 import

## Exports

- `MIME_TYPE_KEY` (상수)
- `A2UI_MIME_TYPE` (상수)
- `DEPRECATED_A2UI_MIME_TYPE` (상수)
- `create_a2ui_part` (함수)
- `is_a2ui_part` (함수)
- `get_a2ui_datapart` (함수)
- `parse_response_to_parts` (함수)
- `stream_response_to_parts` (비동기 제너레이터 함수)

## 상세 명세

### 상수

- `MIME_TYPE_KEY: str = "mimeType"` — DataPart metadata에서 MIME 타입을 나타내는 키.
- `A2UI_MIME_TYPE: str = "application/a2ui+json"` — 현재 A2UI MIME 타입 (v1.0+).
- `DEPRECATED_A2UI_MIME_TYPE: str = "application/json+a2ui"` — 구버전(v0.8, v0.9) 호환을 위한 deprecated MIME 타입.

### `create_a2ui_part(a2ui_data: dict[str, Any], version: Optional[str] = None) -> Part`

A2UI JSON 딕셔너리를 감싸는 `Part(root=DataPart(...))` 객체를 생성한다.

- `version`이 `None`이거나 `"0.8"`, `"0.9"`, `"v0.8"`, `"v0.9"` 중 하나이면 `DEPRECATED_A2UI_MIME_TYPE`을 사용하고, 그 외에는 `A2UI_MIME_TYPE`을 사용한다.
- `DataPart`의 `data` 필드에 `a2ui_data`를, `metadata` 필드에 `{MIME_TYPE_KEY: mime_type}`을 설정한다.

### `is_a2ui_part(part: Part) -> bool`

주어진 `Part`가 A2UI 데이터를 담고 있는지 확인한다.

- `part.root`가 `DataPart` 인스턴스인지 확인한다.
- `part.root.metadata`가 존재하고, `metadata.get(MIME_TYPE_KEY)`가 `A2UI_MIME_TYPE` 또는 `DEPRECATED_A2UI_MIME_TYPE` 중 하나인지 확인한다.
- 두 조건이 모두 충족되면 `True`, 아니면 `False`를 반환한다.

### `get_a2ui_datapart(part: Part) -> Optional[DataPart]`

`Part`에서 A2UI `DataPart`를 추출한다. `is_a2ui_part`로 확인 후 A2UI 파트이면 `part.root`를 반환하고, 아니면 `None`을 반환한다.

### `parse_response_to_parts(content: str, validator: Optional[Any] = None, fallback_text: Optional[str] = None, version: Optional[str] = None) -> List[Part]`

LLM 응답 문자열을 파싱해 `Part` 리스트로 변환한다.

1. `a2ui.parser.parser.parse_response(content)`를 호출해 응답을 파트별로 분해한다(A2UI 딜리미터 기준).
2. 각 파트에 대해:
   - `part.text`가 있으면 `Part(root=TextPart(text=...))`를 추가한다.
   - `part.a2ui_json`이 있으면 `validator`가 있을 경우 먼저 `validator.validate(json_data)`를 호출한다. json_data가 리스트이면 항목별로 `create_a2ui_part`를 호출하고, 단일 딕셔너리이면 하나의 `create_a2ui_part`를 호출한다.
3. 예외 발생 시 `logger.warning`으로 기록하고 해당 파트를 건너뛴다.
4. 파싱 결과가 비어있고 `fallback_text`가 있으면 `Part(root=TextPart(text=fallback_text))`를 추가한다.
5. 최종 `parts` 리스트를 반환한다.

### `stream_response_to_parts(parser: "A2uiStreamParser", token_stream: AsyncIterable[str], version: Optional[str] = None) -> AsyncIterable[Part]`

비동기 제너레이터. LLM 토큰 스트림을 점진적으로 파싱해 `Part`를 yield한다.

1. `token_stream`의 각 토큰에 대해 `parser.process_chunk(token)`을 호출한다. INFO 레벨 로깅으로 수신 토큰과 결과를 기록한다.
2. 반환된 각 응답 파트에 대해:
   - `part.text`가 있으면 `Part(root=TextPart(text=...))`를 yield한다.
   - `part.a2ui_json`이 있으면 리스트 여부에 따라 항목별 혹은 단일 `create_a2ui_part`를 yield한다.

## 동작 흐름

에이전트가 LLM에서 받은 응답(문자열 또는 스트림)을 A2A 프로토콜의 `Part` 목록으로 변환하는 중간 계층 역할을 한다. `parse_response_to_parts`는 동기 완성 응답에, `stream_response_to_parts`는 스트리밍 응답에 사용된다. 두 경우 모두 텍스트 파트와 A2UI JSON 파트를 구분해 적절한 타입으로 래핑한다.
