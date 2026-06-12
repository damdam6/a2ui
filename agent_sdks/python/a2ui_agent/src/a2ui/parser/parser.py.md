# agent_sdks/python/a2ui_agent/src/a2ui/parser/parser.py

## 개요

완성된 LLM 응답 문자열에서 A2UI JSON 블록을 추출하고 파싱하는 모듈이다. 응답 텍스트에서 `<a2ui-json>…</a2ui-json>` 태그 쌍을 정규표현식으로 찾고, 각 블록의 JSON을 마크다운 코드 펜스 제거 및 `parse_and_fix`를 통한 자동 수정을 거쳐 `ResponsePart` 객체 리스트로 변환한다. 스트리밍이 아닌 단발(non-streaming) 응답 파싱 경로를 담당한다.

## 의존성

- 외부 패키지:
  - `re` (표준 라이브러리)
  - `typing` (표준 라이브러리) — `List`, `Optional`, `Any`
- 내부 모듈:
  - [`./response_part.py`](./response_part.py.md) — `ResponsePart`
  - `../schema/constants.py` — `A2UI_OPEN_TAG`, `A2UI_CLOSE_TAG`
  - [`./payload_fixer.py`](./payload_fixer.py.md) — `parse_and_fix`

## Exports

- `has_a2ui_parts` — 함수
- `parse_response` — 함수

(비공개: `_A2UI_BLOCK_PATTERN`, `_sanitize_json_string`)

## 상세 명세

### `_A2UI_BLOCK_PATTERN`

모듈 레벨 컴파일된 정규표현식 패턴이다. `A2UI_OPEN_TAG`와 `A2UI_CLOSE_TAG` 사이의 모든 내용을 `re.DOTALL` 플래그로 캡처한다 (줄바꿈 포함). `re.escape`로 태그를 이스케이프한 뒤 `(.*?)` 비탐욕적 캡처 그룹을 사용한다.

---

### `has_a2ui_parts(content: str) -> bool`

**목적:** 문자열이 A2UI 태그 쌍을 모두 포함하는지 빠르게 확인하는 프리플라이트 검사다.

**동작:** `A2UI_OPEN_TAG in content`와 `A2UI_CLOSE_TAG in content`를 모두 `and`로 결합하여 반환한다. 정규표현식 없이 단순 부분 문자열 검색을 사용한다.

---

### `_sanitize_json_string(json_string: str) -> str`

**목적:** JSON 문자열 전처리 단계로, LLM이 JSON 주변에 마크다운 코드 펜스를 추가하는 경우를 처리한다.

**동작 단계:**
1. `json_string.strip()`으로 앞뒤 공백을 제거한다.
2. 문자열이 `` ```json ``으로 시작하면 해당 접두사(7자)를 제거한다. 그렇지 않고 `` ``` ``으로만 시작하면 3자를 제거한다.
3. 문자열이 `` ``` ``으로 끝나면 마지막 3자를 제거한다.
4. 결과를 다시 `strip()`하여 반환한다.

---

### `parse_response(content: str) -> List[ResponsePart]`

**목적:** 완전한 LLM 응답 문자열을 파싱하여 `ResponsePart` 객체 리스트로 반환한다.

**매개변수:**
- `content: str` — 파싱할 원시 LLM 응답 전체 문자열

**반환값:** `List[ResponsePart]` — 텍스트 파트와 A2UI JSON 파트를 순서대로 담은 리스트

**예외:**
- `ValueError` — A2UI 태그가 없거나 JSON 파트가 비어 있을 때, 또는 `parse_and_fix`가 파싱에 실패했을 때 발생

**동작 단계:**
1. `_A2UI_BLOCK_PATTERN.finditer(content)`로 모든 A2UI 블록 매치를 리스트로 수집한다.
2. 매치가 없으면 `ValueError`를 raise한다 (태그 미발견 메시지 포함).
3. `last_end = 0`으로 초기화한다. 각 매치에 대해 반복한다:
   a. 매치 시작 위치(`start`) 이전 `content[last_end:start]`를 `strip()`하여 `text_part`를 만든다.
   b. `match.group(1)`로 태그 사이의 JSON 문자열을 꺼낸다.
   c. `_sanitize_json_string`으로 마크다운 코드 펜스를 제거한다.
   d. 결과가 비어 있으면 `ValueError("A2UI JSON part is empty.")`를 raise한다.
   e. `parse_and_fix(json_string_cleaned)`로 JSON을 파싱 및 자동 수정한다.
   f. `ResponsePart(text=text_part, a2ui_json=json_data)`를 리스트에 추가한다.
   g. `last_end = end`로 커서를 매치 끝으로 이동한다.
4. 마지막 매치 이후 남은 `content[last_end:]`를 `strip()`하여 `trailing_text`를 만든다. 비어 있지 않으면 `ResponsePart(text=trailing_text, a2ui_json=None)`을 추가한다.
5. 완성된 `response_parts` 리스트를 반환한다.

## 동작 흐름

호출자는 완성된 LLM 응답을 `parse_response`에 전달한다. 함수는 정규표현식으로 모든 A2UI 블록을 한 번에 찾은 뒤, 블록 사이의 텍스트와 블록 내 JSON을 번갈아 추출한다. 각 JSON은 마크다운 정리 → `parse_and_fix` 파싱/수정을 거쳐 `ResponsePart`로 패키징된다. 결과 리스트는 원래 응답의 선형 순서를 유지한다.
