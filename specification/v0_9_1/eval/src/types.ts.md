# specification/v0_9_1/eval/src/types.ts

## 개요

이 파일은 평가 파이프라인의 각 단계를 거치면서 결과 객체가 갖는 타입을 정의한다. 생성(Generation) → 검증(Validation) → 평가(Evaluation)의 세 단계에 대응하는 세 가지 인터페이스가 확장 관계로 연결되어 있으며, 파이프라인 전반에서 결과를 전달하고 저장하는 공통 형식을 제공한다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈
- [`./prompts`](./prompts.ts.md) — `TestPrompt` (타입 참조)

## Exports

| 이름 | 종류 |
|---|---|
| `GeneratedResult` | 인터페이스 |
| `ValidatedResult` | 인터페이스 |
| `IssueSeverity` | 타입 별칭 |
| `EvaluatedResult` | 인터페이스 |

## 상세 명세

### `GeneratedResult` 인터페이스 (export)

LLM 생성 단계 이후 한 번의 실행 결과를 나타낸다.

| 필드 | 타입 | 설명 |
|---|---|---|
| `modelName` | `string` | 생성에 사용된 모델의 식별자 |
| `prompt` | `TestPrompt` | 사용된 테스트 프롬프트 객체 |
| `runNumber` | `number` | 동일 프롬프트에 대한 반복 실행 번호 |
| `rawText` | `string` (선택) | 모델이 반환한 원시 텍스트 응답 |
| `components` | `any[]` (선택) | `rawText`에서 파싱된 JSON 오브젝트 배열 |
| `latency` | `number` | 요청 전송부터 응답 완료까지의 소요 시간 (밀리초) |
| `error` | `any` (선택) | 생성 실패 시 오류 객체 |

### `ValidatedResult` 인터페이스 (export)

`GeneratedResult`를 확장하며 스키마 검증 단계 결과를 추가한다.

| 추가 필드 | 타입 | 설명 |
|---|---|---|
| `validationErrors` | `string[]` | AJV 및 커스텀 검증에서 발견된 오류 메시지 배열. 오류 없음이면 빈 배열 |

### `IssueSeverity` 타입 별칭 (export)

평가 단계에서 발견된 문제의 심각도를 나타내는 유니온 타입.

가능한 값: `'minor'` | `'significant'` | `'critical'` | `'criticalSchema'`

- `'criticalSchema'`: 스키마 검증 실패로 판정된 가장 심각한 오류
- `'critical'`: 기능적으로 치명적인 오류
- `'significant'`: 중요하지만 치명적이지 않은 문제
- `'minor'`: 사소한 문제

### `EvaluatedResult` 인터페이스 (export)

`ValidatedResult`를 확장하며 LLM 기반 평가 단계 결과를 추가한다.

| 추가 필드 | 타입 | 설명 |
|---|---|---|
| `evaluationResult` | 객체 (선택) | 평가 결과 전체. 평가가 수행되지 않았으면 `undefined` |
| `evaluationResult.pass` | `boolean` | 전체 합격/불합격 판정 |
| `evaluationResult.reason` | `string` | 판정 이유 서술 |
| `evaluationResult.issues` | `{issue: string; severity: IssueSeverity}[]` (선택) | 발견된 개별 문제 목록 |
| `evaluationResult.overallSeverity` | `IssueSeverity` (선택) | 전체 심각도 요약 |
| `evaluationResult.evalPrompt` | `string` (선택) | 평가에 사용된 프롬프트 텍스트 (디버깅용) |

## 동작 흐름

파이프라인의 첫 번째 단계(생성)는 `GeneratedResult` 배열을 출력한다. 두 번째 단계(검증)는 이 배열을 받아 각 항목에 `validationErrors`를 추가한 `ValidatedResult` 배열을 반환한다. 세 번째 단계(평가)는 `ValidatedResult` 배열을 받아 `evaluationResult`를 추가한 `EvaluatedResult` 배열을 반환한다. 각 단계의 함수가 이전 단계 결과를 스프레드 연산으로 복사하고 새 필드를 덧붙이는 방식으로 타입이 누적 확장된다.
