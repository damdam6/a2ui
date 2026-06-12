# agent_sdks/python/a2ui_agent/src/a2ui/parser/response_part.py

## 개요

LLM 응답의 한 단위를 표현하는 데이터 클래스 `ResponsePart`를 정의한다. 응답은 일반 대화형 텍스트와 A2UI JSON 데이터의 두 부분으로 구성될 수 있으며, 이 클래스는 그 쌍을 단일 객체로 캡슐화한다. 파서와 스트리밍 모듈 전반에서 파싱 결과의 기본 전달 단위로 사용된다.

## 의존성

- 외부 패키지:
  - `dataclasses` (표준 라이브러리) — `@dataclass` 데코레이터
  - `typing` (표준 라이브러리) — `Any`, `Optional`
- 내부 모듈: 없음

## Exports

- `ResponsePart` — 클래스 (dataclass)

## 상세 명세

### `ResponsePart`

`@dataclass` 데코레이터로 선언된 데이터 클래스다.

**필드:**

| 필드 | 타입 | 기본값 | 설명 |
|------|------|--------|------|
| `text` | `str` | `""` | 대화형 텍스트 부분. A2UI 블록 앞뒤의 일반 텍스트가 저장된다. 값이 없을 때는 빈 문자열이다. |
| `a2ui_json` | `Optional[Any]` | `None` | 파싱된 A2UI JSON 데이터. A2UI 메시지를 포함할 경우 항상 딕셔너리의 리스트 형태다. 텍스트만 있는 trailing 파트의 경우 `None`이다. |

**생명주기:** `@dataclass`이므로 `__init__`, `__repr__`, `__eq__`가 자동 생성된다. 별도의 메서드나 상속은 없다.

## 동작 흐름

모듈 임포트 시 `ResponsePart` 클래스가 정의된다. 파서([`parser.py`](./parser.py.md))와 스트리밍 파서([`streaming.py`](./streaming.py.md))는 파싱 결과를 `ResponsePart` 인스턴스 리스트로 반환한다. 텍스트 전용 파트는 `a2ui_json=None`, A2UI 데이터 파트는 `text=""`이고 `a2ui_json`에 메시지 리스트가 들어간다. 두 값이 동시에 채워질 수도 있다 (텍스트 + JSON 쌍).
