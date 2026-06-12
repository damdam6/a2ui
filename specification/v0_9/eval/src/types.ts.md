# specification/v0_9/eval/src/types.ts

## 개요

평가 파이프라인의 각 단계(생성 → 유효성 검사 → 평가)를 거치며 데이터가 어떻게 확장되는지를 나타내는 타입 정의 파일이다. 세 인터페이스가 계층적으로 확장되는 구조로, `GeneratedResult`가 기반이 되고 `ValidatedResult`가 이를 상속하며 `EvaluatedResult`가 최종 단계를 표현한다. `IssueSeverity` 유니언 타입으로 이슈 심각도를 분류한다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈
- [`./prompts`](./prompts.ts.md) — `TestPrompt` 타입 (GeneratedResult 필드로 사용)

## Exports

| 이름 | 종류 | 설명 |
|------|------|------|
| `GeneratedResult` | 인터페이스 | LLM 생성 결과 |
| `ValidatedResult` | 인터페이스 | 스키마 검증 결과 (GeneratedResult 확장) |
| `IssueSeverity` | 타입 (유니언) | 이슈 심각도 등급 |
| `EvaluatedResult` | 인터페이스 | LLM 평가 결과 (ValidatedResult 확장) |

## 상세 명세

### `GeneratedResult` 인터페이스 (export)

1단계: LLM 호출로 생성된 원시 결과.

| 필드 | 타입 | 필수 여부 | 설명 |
|------|------|-----------|------|
| `modelName` | `string` | 필수 | 결과를 생성한 모델의 이름 |
| `prompt` | `TestPrompt` | 필수 | 사용된 테스트 프롬프트 |
| `runNumber` | `number` | 필수 | 동일 프롬프트·모델 조합에서의 실행 번호 |
| `rawText` | `string` | 선택 | LLM이 반환한 원시 텍스트 응답 |
| `components` | `any[]` | 선택 | 파싱된 JSON 컴포넌트 배열 |
| `latency` | `number` | 필수 | 요청-응답 소요 시간(ms) |
| `error` | `any` | 선택 | 생성 중 발생한 에러 객체 |

### `ValidatedResult` 인터페이스 (export)

2단계: `GeneratedResult`를 확장하며 스키마 검증 결과를 추가.

| 추가 필드 | 타입 | 설명 |
|-----------|------|------|
| `validationErrors` | `string[]` | AJV 및 커스텀 검증에서 수집된 에러 메시지 목록. 통과 시 빈 배열 |

### `IssueSeverity` 타입 (export)

`'minor' | 'significant' | 'critical' | 'criticalSchema'` 4단계 유니언 타입. 각 값의 의미:
- `'minor'` — 경미한 문제
- `'significant'` — 중요한 문제
- `'critical'` — 심각한 문제
- `'criticalSchema'` — 스키마 검증 실패로 인한 치명적 문제

### `EvaluatedResult` 인터페이스 (export)

3단계: `ValidatedResult`를 확장하며 LLM 기반 평가 결과를 추가.

| 추가 필드 | 타입 | 설명 |
|-----------|------|------|
| `evaluationResult` | 객체 (선택) | LLM 평가기의 판정 결과 |
| `evaluationResult.pass` | `boolean` | 통과/실패 여부 |
| `evaluationResult.reason` | `string` | 판정 이유 설명 |
| `evaluationResult.issues` | `{issue: string; severity: IssueSeverity}[]` | 선택, 개별 이슈 목록 |
| `evaluationResult.overallSeverity` | `IssueSeverity` | 선택, 전체 심각도 |
| `evaluationResult.evalPrompt` | `string` | 선택, 평가에 사용된 프롬프트 텍스트 |

## 동작 흐름

평가 파이프라인은 데이터를 단계별로 변환하며 타입을 확장한다. 생성 단계에서 `GeneratedResult` 객체를 만들고, 검증 단계에서 스프레드 연산자로 `ValidatedResult`로 변환(`{...result, validationErrors: errors}`)하며, 최종 평가 단계에서 `EvaluatedResult`로 확장한다. 이 단방향 확장 구조 덕분에 각 단계에서 이전 단계의 모든 데이터를 보존한다.
