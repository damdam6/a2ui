# agent_sdks/python/a2ui_agent/src/a2ui/parser/payload_fixer.py

## 개요

LLM이 생성한 JSON 문자열의 일반적인 형식 오류를 자동으로 수정하고 파싱하는 모듈이다. 스마트 인용 부호(curly quotes)를 표준 인용 부호로 교체하고, JSON 파싱 실패 시 후행 쉼표(trailing comma)를 제거하는 두 단계 자동 복구 전략을 제공한다. 공개 함수 `parse_and_fix`가 진입점이며, 나머지 함수들은 비공개 헬퍼다.

## 의존성

- 외부 패키지:
  - `json` (표준 라이브러리)
  - `logging` (표준 라이브러리)
  - `re` (표준 라이브러리)
  - `typing` (표준 라이브러리) — `Any`, `Dict`, `List`
- 내부 모듈: 없음

## Exports

- `parse_and_fix` — 함수

## 상세 명세

### 모듈 레벨

`logger = logging.getLogger(__name__)`로 모듈 전용 로거 인스턴스를 생성한다.

---

### `parse_and_fix(payload: str) -> List[Dict[str, Any]]`

**목적:** 원시 JSON 문자열을 검증하고 자동 수정하여 파싱된 페이로드를 반환한다.

**동작 단계:**
1. `_normalize_smart_quotes(payload)`를 호출하여 curly quote 문자를 표준 ASCII 인용 부호로 교체한 `normalized_payload`를 얻는다.
2. `_parse(normalized_payload)`를 시도한다. 성공하면 결과를 즉시 반환한다.
3. `json.JSONDecodeError` 또는 `ValueError`가 발생하면 `logger.warning`으로 초기 실패를 기록한다.
4. `_remove_trailing_commas(normalized_payload)`로 후행 쉼표를 제거한 `updated_payload`를 만든다.
5. `_parse(updated_payload)`를 재시도한다. 이 호출이 실패하면 예외가 호출자에게 전파된다.

---

### `_parse(payload: str) -> List[Dict[str, Any]]`

**목적:** JSON 문자열을 파싱하고, 단일 객체일 경우 리스트로 감싼다.

**동작 단계:**
1. `json.loads(payload)`로 파싱을 시도한다.
2. 결과가 `list`가 아니면 `logger.info`로 감싸는 사실을 기록하고 단일 원소 리스트 `[a2ui_json]`으로 변환한다.
3. `json.JSONDecodeError` 발생 시 `logger.error`로 기록 후 `ValueError`로 재포장(wrapping)하여 raise한다.
4. 정상적인 경우 리스트를 반환한다.

---

### `_normalize_smart_quotes(json_str: str) -> str`

**목적:** LLM이 생성하는 curly quote 유니코드 문자를 표준 직선 인용 부호로 교체한다.

**교체 매핑:**
- `“` (`"`) → `"`
- `”` (`"`) → `"`
- `‘` (`'`) → `'`
- `’` (`'`) → `'`

연속적인 `.replace()` 체인으로 구현되며, 수정된 새 문자열을 반환한다.

---

### `_remove_trailing_commas(json_str: str) -> str`

**목적:** JSON에서 유효하지 않은 후행 쉼표를 제거한다.

**동작 단계:**
1. 정규표현식 `r",(?=\s*[\]}])"` 으로 닫는 괄호(`]`) 또는 닫는 중괄호(`}`) 직전의 쉼표(공백 포함)를 모두 빈 문자열로 치환한다.
2. 원본과 수정 결과가 다를 경우 `logger.warning`으로 자동 수정 적용 사실을 기록한다.
3. 수정된(또는 원본과 동일한) 문자열을 반환한다.

## 동작 흐름

[`parser.py`](./parser.py.md)의 `parse_response` 함수가 A2UI 태그 내부 JSON 문자열을 추출한 뒤 `parse_and_fix`를 호출한다. `parse_and_fix`는 먼저 curly quote 정규화를 수행한 다음, 1차 파싱을 시도하고, 실패 시 trailing comma 제거 후 2차 파싱을 시도하는 순서로 실행된다. 두 번의 수정을 거쳐도 파싱에 실패하면 `ValueError`가 호출 스택 위로 전파된다.
