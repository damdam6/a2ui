# specification/v0_9/eval/src/utils.ts

## 개요

LLM 응답 텍스트에서 JSON 데이터를 추출하는 유틸리티 함수를 제공하는 파일이다. 마크다운 코드 펜스(` ```json ... ``` `)에 감싸인 JSON 블록을 정규식으로 찾아내고, 각 블록을 단일 JSON 객체 또는 JSONL(줄 단위 JSON) 형식으로 파싱한다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈
없음

## Exports

| 이름 | 종류 | 설명 |
|------|------|------|
| `extractJsonFromMarkdown` | 함수 | 마크다운 텍스트에서 JSON 객체 배열을 추출 |

## 상세 명세

### `extractJsonFromMarkdown(markdown: string): any[]`

마크다운 문자열에서 JSON 블록을 모두 찾아 파싱된 객체 배열로 반환한다.

**매개변수**
- `markdown: string` — LLM이 반환한 원시 텍스트 (마크다운 형식 포함 가능)

**반환 타입**
- `any[]` — 파싱 성공한 JSON 객체들의 배열. 파싱 가능한 항목이 없으면 빈 배열 반환.

**동작 단계**

1. 정규식 `` /```json\s*([\s\S]*?)\s*```/g `` 를 사용하여 `markdown` 전체에서 ` ```json ` 펜스 블록을 모두 탐색한다(`matchAll`). 각 매치의 캡처 그룹 `[1]`이 블록 내용이다.
2. 각 블록에 대해 내용 문자열을 `trim()`한 뒤 `JSON.parse`를 시도한다.
   - 성공하면 파싱 결과를 `results` 배열에 추가한다.
   - 실패하면 JSONL 형식을 시도한다: 내용을 줄(`\n`)로 분리하여 비어 있지 않은 각 줄에 대해 `JSON.parse`를 시도하고, 성공한 줄만 `results`에 추가한다. 파싱 실패 줄은 무시한다.
3. 모든 블록을 처리한 뒤 `results` 배열을 반환한다.

**에러 처리**
- 최상위 `JSON.parse` 실패는 catch하여 JSONL 재시도로 넘어간다.
- JSONL의 개별 줄 파싱 실패는 catch하여 조용히 무시한다.
- 두 시도 모두 실패한 블록의 내용은 결과에 포함되지 않는다.

## 동작 흐름

LLM 응답이 들어오면 이 함수를 호출하여 JSON 배열로 변환한다. 반환된 배열의 각 요소는 `createSurface`, `updateComponents`, `updateDataModel`, `deleteSurface` 중 하나의 A2UI 메시지 객체로 취급되며, 이후 `Validator`의 `run` 메서드에서 스키마 검증을 받는다.
