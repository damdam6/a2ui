# agent_sdks/python/a2ui_agent/src/a2ui/schema/common_modifiers.py

## 개요

JSON 스키마를 변환하는 공통 수정 함수(modifier)를 제공한다. 현재는 JSON 스키마에서 `additionalProperties: false` 제약을 재귀적으로 제거하는 단일 함수만 포함하고 있다. 이 함수는 `A2uiSchemaManager`의 `schema_modifiers` 목록에 등록하여 로드된 스키마를 LLM에 전달하기 전에 유연성을 높이는 용도로 사용된다.

## 의존성

없음 (표준 라이브러리 또는 외부 패키지 미사용).

## Exports

- 함수: `remove_strict_validation`

## 상세 명세

### 함수 `remove_strict_validation(schema: Any) -> Any`

JSON 스키마 객체(`dict`, `list`, 또는 원시 값)를 재귀적으로 순회하며 `additionalProperties` 키의 값이 정확히 `False`인 경우 해당 키를 제거하여 새 객체를 반환한다.

매개변수:
- `schema: Any` — 처리할 JSON 스키마 (dict, list, 또는 스칼라 값)

반환 타입: 입력과 동일한 구조의 새 객체

로직:
1. `schema`가 `dict`이면: 모든 키-값 쌍에 대해 재귀 적용하여 새 dict를 생성한다. 생성된 dict에 `'additionalProperties'` 키가 있고 그 값이 `False`이면 해당 키를 삭제한다. 수정된 dict를 반환한다.
2. `schema`가 `list`이면: 각 항목에 재귀 적용한 새 list를 반환한다.
3. 그 외(스칼라): `schema`를 그대로 반환한다.

경계 케이스: `additionalProperties` 값이 dict(예: 빈 스키마 `{}`) 등 `False`가 아닌 경우에는 제거하지 않는다.

## 동작 흐름

입력 스키마 트리를 완전히 복사하면서 `additionalProperties: false` 패턴만 제거한다. 원본 스키마는 변경되지 않으며, 항상 새 객체를 반환한다.
