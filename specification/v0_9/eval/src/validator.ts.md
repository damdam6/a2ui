# specification/v0_9/eval/src/validator.ts

## 개요

LLM이 생성한 A2UI 메시지 배열을 AJV JSON 스키마 검증과 커스텀 참조 무결성 검증으로 검사하는 `Validator` 클래스를 제공한다. AJV를 사용하여 `server_to_client.json` 스키마 및 컴포넌트별 카탈로그 스키마에 대한 정밀 검증을 수행하며, `oneOf`/`anyOf` 노이즈를 억제하기 위해 컴포넌트 경로를 직접 추적한 뒤 타깃 검증을 수행한다. 검증 실패 시 YAML 파일로 결과를 저장한다.

## 의존성

### 외부 패키지
- `ajv/dist/2020` — `Ajv` (JSON Schema Draft 2020-12 검증기)
- `ajv-formats` — `addFormats` (date, uri 등 포맷 지원)
- `fs` (Node.js 내장) — 실패 결과 파일 저장
- `path` (Node.js 내장) — 출력 디렉터리 경로 조합
- `js-yaml` — `yaml` (실패 결과 YAML 직렬화)

### 저장소 내부 모듈
- [`./types`](./types.ts.md) — `GeneratedResult`, `ValidatedResult`, `IssueSeverity`
- [`./logger`](./logger.ts.md) — `logger`

## Exports

| 이름 | 종류 | 설명 |
|------|------|------|
| `Validator` | 클래스 | 스키마 + 커스텀 검증 실행기 |

## 상세 명세

### `Validator` 클래스 (export)

#### 필드

| 필드 | 타입 | 접근성 | 설명 |
|------|------|--------|------|
| `ajv` | `Ajv` | private | `{allErrors: true, strict: false}` 설정의 AJV 인스턴스 |
| `validateFn` | `any` | private | `server_to_client.json` 루트 스키마에 대한 컴파일된 검증 함수 |
| `basicFunctions` | `Set<string>` | private | 카탈로그에서 로드한 허용 함수 이름 집합 |
| `schemas` | `Record<string, any>` | private (constructor 매개변수) | 파일명을 키로 하는 스키마 맵 |
| `outputDir` | `string \| undefined` | private (constructor 매개변수) | 실패 결과 저장 디렉터리 |

#### `constructor(schemas: Record<string, any>, outputDir?: string)`

1. `{allErrors: true, strict: false}`로 `Ajv` 인스턴스를 생성하고 `addFormats`를 적용한다.
2. `schemas` 맵의 모든 항목을 `ajv.addSchema(schema, name)`으로 등록한다.
3. `ajv.getSchema('https://a2ui.org/specification/v0_9/server_to_client.json')`으로 루트 검증 함수를 `validateFn`에 저장한다.
4. `schemas['catalogs/basic/catalog.json']`의 `.functions` 객체 키를 `basicFunctions` Set에 추가한다. 로드된 함수가 없으면 경고 로그를 남긴다.

#### `async run(results: GeneratedResult[]): Promise<ValidatedResult[]>`

Phase 2 검증을 순차적으로 실행하는 공개 메서드(CPU 바운드이므로 직렬 처리).

각 `GeneratedResult`에 대해:
1. `result.error`가 있거나 `result.components`가 없으면 `validationErrors: []`를 붙여 통과시킨다.
2. 그렇지 않으면 `components` 배열의 각 메시지에 대해 AJV 스마트 검증을 수행한다.
   - 메시지 최상위 키가 `createSurface`이면 `$defs/CreateSurfaceMessage`, `updateComponents`이면 `$defs/UpdateComponentsMessage`, `updateDataModel`이면 `$defs/UpdateDataModelMessage`, `deleteSurface`이면 `$defs/DeleteSurfaceMessage`에 대해 검증한다.
   - 해당 키가 없으면 `validateFn`(루트 스키마) 폴백 검증을 수행한다.
   - 검증 실패 시 타깃 에러 수집 절차를 시작한다(아래 설명).
3. `validateCustom`으로 커스텀 검증을 추가 수행한다.
4. 에러 발생 시 `outputDir`가 있으면 `saveFailure`를 호출한다.
5. `{...result, validationErrors: errors}`를 결과 배열에 추가한다.
6. 최종 통과/실패 카운트를 로그로 남기고 반환한다.

**타깃 에러 수집 절차** (AJV 검증 실패 시):

1. `traverse` 재귀 함수로 메시지 내 모든 객체를 순회하며 `component` 문자열 프로퍼티를 가진 객체의 인스턴스 경로를 `pathToObject` Map에 수집한다(순환 참조 방지를 위해 `visited` Set 사용).
2. 수집된 각 컴포넌트 경로에 대해 `catalogs/basic/catalog.json#/components/${componentName}` 스키마로 타깃 검증을 수행한다.
   - 스키마를 찾지 못하면(catch) 환각 컴포넌트 에러를 `targetedErrors`에 추가하고 해당 경로를 처리 완료로 마킹한다.
   - 검증 실패 시 각 에러의 `instancePath`에 기반 경로를 프리펜드하여 추가한다.
3. 원본 AJV 에러에서 `oneOf`/`anyOf` 키워드 에러를 제거하고, 이미 타깃 처리된 경로 하위의 에러도 제거하는 필터링을 수행한다.
4. 필터링된 원본 에러와 `targetedErrors`를 합쳐 중복 제거 후 포맷팅한다.
   - 각 에러 메시지 포맷: `${instancePath}(${componentType}) ${message} (\n    key: value\n  )` 형태로 `params`를 포함한다.

#### `private saveFailure(result: GeneratedResult, errors: string[]): void`

`outputDir/output-${modelName}/details/${promptName}.${runNumber}.failed.yaml` 경로에 검증 실패 정보를 YAML로 저장한다. 모델 이름에서 `/`와 `:`는 `_`로 치환한다. 저장 데이터 구조: `{pass: false, reason: 'Schema validation failure', issues: [{issue, severity: 'criticalSchema'}], overallSeverity: 'criticalSchema'}`.

#### `private validateCustom(messages: any[], errors: string[]): void`

메시지 스트림 전체에 대한 커스텀 검증.

1. 모든 메시지를 순회하며 메시지 유형별로 분기한다.
   - `updateComponents`: `surfaceId`가 이미 `createSurface`로 선언됐는지 확인하고, `validateUpdateComponents`를 호출하며, `id: 'root'` 컴포넌트 존재 여부를 추적한다.
   - `createSurface`: `validateCreateSurface`를 호출하고 `surfaceId`를 `createdSurfaces` Set에 추가한다.
   - `updateDataModel`: `validateUpdateDataModel`을 호출한다.
   - `deleteSurface`: `validateDeleteSurface`를 호출한다.
   - 그 외: 알 수 없는 메시지 타입 에러를 추가한다.
2. `updateComponents`가 있는데 `root` 컴포넌트가 없으면 에러를 추가한다.
3. `validateFunctionCalls`로 함수 호출 검증을 수행한다.

#### `private validateFunctionCalls(root: any, errors: string[]): void`

메시지 트리를 재귀적으로 탐색하며 `FunctionCall` 구조(`call` 문자열 프로퍼티와 총 2~3개의 키를 가진 객체)를 감지한다. `basicFunctions`에 있는 함수명이면 통과(더미 검증)한다. 알 수 없는 함수는 이 메서드에서는 무시하고 AJV 스키마 검증에 위임한다. 배열과 객체를 모두 재귀적으로 탐색한다.

#### `private validateCreateSurface(data: any, errors: string[]): void`

`createSurface` 페이로드 검증. `surfaceId` 필수, `catalogId` 필수, 허용 키는 `['surfaceId', 'catalogId']`만 허용하며 추가 키가 있으면 에러를 추가한다.

#### `private validateDeleteSurface(data: any, errors: string[]): void`

`deleteSurface` 페이로드 검증. `surfaceId` 필수, 허용 키는 `['surfaceId']`만 허용한다.

#### `private validateUpdateComponents(data: any, errors: string[]): void`

`updateComponents` 페이로드 검증.
1. `surfaceId` 필수, `components` 배열 필수.
2. 컴포넌트 ID 중복 감지.
3. `component` 타입별 AJV 검증: `catalogs/basic/catalog.json#/components/${componentType}` 스키마로 각 컴포넌트를 검증한다.
4. `validateComponent`로 각 컴포넌트의 참조 무결성을 검사한다.

#### `private validateUpdateDataModel(data: any, errors: string[]): void`

`updateDataModel` 페이로드의 커스텀 검증. v0.9에서는 `op` 필드가 제거되어 스키마 검증에 전적으로 의존하므로 현재 구현 본문은 비어 있다.

#### `private validateComponent(component: any, allIds: Set<string>, errors: string[]): void`

개별 컴포넌트의 참조 무결성 검사.

1. `id` 누락이면 에러 후 반환, `component` 타입 누락이면 에러 후 반환.
2. `componentType` 스위치:
   - `'Row'`, `'Column'`, `'List'`: `children`이 배열이면 각 ID를, 객체이면 `children.componentId`를 `checkRefs`로 확인한다.
   - `'Card'`: `child` ID를 확인한다.
   - `'Tabs'`: `tabs` 배열의 각 탭의 `child` ID를 확인한다.
   - `'Modal'`: `trigger`와 `content` ID를 확인한다.
   - `'Button'`: `child` ID를 확인한다.
3. 내부 `checkRefs` 함수: ID가 `allIds` Set에 없으면 "references non-existent component ID" 에러를 추가한다.

## 동작 흐름

1. 애플리케이션 초기화 시 스키마 맵과 출력 디렉터리를 인자로 `Validator` 인스턴스를 생성한다.
2. 생성 단계 결과(`GeneratedResult[]`)를 `validator.run(results)`에 전달한다.
3. `run`은 각 결과를 순차적으로 처리하여 AJV 스마트 검증 → 타깃 에러 수집 → 커스텀 참조 무결성 검증 순으로 실행한다.
4. 에러가 있는 결과는 YAML로 저장되고, 모든 결과는 `validationErrors` 필드가 추가된 `ValidatedResult[]`로 반환된다.
5. 반환된 배열은 다음 단계인 LLM 기반 평가 단계(`EvaluatedResult`)에 입력된다.
