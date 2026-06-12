# specification/v0_9_1/eval/src/utils.ts

## 개요

이 파일은 LLM 응답 텍스트에서 JSON 데이터를 추출하는 단일 유틸리티 함수를 제공한다. 마크다운 코드 펜스(` ```json ... ``` `)로 감싸진 블록을 찾아 각 블록의 내용을 단일 JSON 오브젝트 또는 JSONL(줄 단위 JSON) 형식으로 파싱하여 파싱된 오브젝트 배열로 반환한다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈
없음

## Exports

| 이름 | 종류 |
|---|---|
| `extractJsonFromMarkdown` | 함수 |

## 상세 명세

### `extractJsonFromMarkdown(markdown: string): any[]`

마크다운 문자열에서 모든 JSON 코드 블록을 찾아 파싱된 JavaScript 값의 배열을 반환한다.

**매개변수**
- `markdown: string` — LLM이 반환한 원시 마크다운 텍스트

**반환 타입**
- `any[]` — 파싱 성공한 JSON 값들의 배열. 파싱 불가능한 내용은 조용히 무시된다.

**동작 단계**
1. 정규식 `` /```json\s*([\s\S]*?)\s*```/g ``으로 마크다운 전체에서 모든 JSON 코드 블록을 매치한다. `g` 플래그와 `matchAll`을 사용하여 여러 블록을 모두 찾는다.
2. 각 매치(`match`)에서 캡처 그룹 `match[1]`(코드 블록 본문)을 꺼내 `trim()`한다.
3. 먼저 전체 내용을 `JSON.parse`로 단일 JSON 오브젝트로 파싱을 시도한다. 성공하면 `results`에 추가한다.
4. 단일 파싱이 실패하면(`catch`), JSONL 방식으로 폴백한다: 내용을 `'\n'`으로 줄 단위로 분리하고, 빈 줄이 아닌 각 줄에 대해 개별적으로 `JSON.parse`를 시도한다. 각 줄의 파싱 성공 시 `results`에 추가하고, 실패(`catch e2`) 시 조용히 무시한다.
5. 모든 블록 처리 후 `results` 배열을 반환한다.

**경계 케이스**
- 마크다운에 JSON 코드 블록이 없으면 빈 배열 `[]`를 반환한다.
- 코드 블록 내용이 완전히 유효하지 않은 JSON이고 JSONL로도 파싱되지 않으면 해당 블록은 건너뛴다.
- 한 코드 블록 안에 여러 JSON 오브젝트가 줄바꿈으로 구분된 경우(JSONL) 각각이 개별 항목으로 `results`에 추가된다.

## 동작 흐름

생성 단계에서 LLM 응답의 `rawText`를 이 함수에 전달하면 파싱된 메시지 오브젝트 배열(`components`)을 얻는다. 이 배열이 `GeneratedResult.components`에 저장되어 이후 검증·평가 단계로 전달된다.
