# specification/v0_9_1/eval/src/validator.ts

## 개요

이 파일은 LLM이 생성한 a2ui 프로토콜 메시지들을 JSON 스키마(AJV)와 커스텀 비즈니스 로직으로 검증하는 `Validator` 클래스를 정의한다. AJV를 사용한 구조 검증과, 컴포넌트 ID 참조 무결성·메시지 순서 등을 확인하는 커스텀 검증을 이중으로 수행한다. 검증 실패 시 YAML 형식의 실패 파일을 출력 디렉토리에 저장하는 기능도 포함한다.

## 의존성

### 외부 패키지
- `ajv/dist/2020` — `Ajv` (JSON Schema Draft 2020-12 검증기)
- `ajv-formats` — `addFormats` (AJV에 `format` 키워드 지원 추가)
- `fs` — Node.js 내장 파일 시스템 모듈
- `path` — Node.js 내장 경로 모듈
- `js-yaml` — `yaml` (YAML 직렬화)

### 저장소 내부 모듈
- [`./types`](./types.ts.md) — `GeneratedResult`, `ValidatedResult`, `IssueSeverity`
- [`./logger`](./logger.ts.md) — `logger`

## Exports

| 이름 | 종류 |
|---|---|
| `Validator` | 클래스 |

## 상세 명세

### `Validator` 클래스 (export)

#### 생성자: `constructor(schemas: Record<string, any>, outputDir?: string)`

**매개변수**
- `schemas`: 스키마 파일명(키) → 스키마 오브젝트(값) 매핑. 호출자(`index.ts`)가 파일 시스템에서 읽어 전달한다.
- `outputDir`: 검증 실패 파일을 저장할 디렉토리 경로 (선택).

**초기화 단계**
1. `new Ajv({allErrors: true, strict: false})`로 AJV 인스턴스를 생성한다. `allErrors: true`는 첫 오류에서 멈추지 않고 모든 오류를 수집하며, `strict: false`는 알 수 없는 키워드를 허용한다.
2. `addFormats(this.ajv)`로 `format` 키워드(예: `uri`, `date-time` 등)를 활성화한다.
3. `schemas` 오브젝트의 모든 항목을 `ajv.addSchema(schema, name)`으로 등록한다.
4. `ajv.getSchema('https://a2ui.org/specification/v0_9/server_to_client.json')`로 최상위 메시지 유니온 스키마의 검증 함수를 `validateFn`에 저장한다.
5. `schemas['catalogs/basic/catalog.json']`에서 `functions` 키의 모든 함수 이름을 `basicFunctions: Set<string>`에 로드한다. 함수가 하나도 없으면 `logger.warn`으로 경고를 출력한다.

#### 필드

| 필드 | 타입 | 접근 | 설명 |
|---|---|---|---|
| `ajv` | `Ajv` | private | AJV 검증기 인스턴스 |
| `validateFn` | `any` | private | 최상위 스키마 검증 함수 (폴백용) |
| `basicFunctions` | `Set<string>` | private | 허용된 기본 함수 이름 집합 |
| `schemas` | `Record<string, any>` | private (생성자 인자) | 등록된 스키마 전체 |
| `outputDir` | `string \| undefined` | private (생성자 인자) | 실패 파일 저장 경로 |

---

#### `async run(results: GeneratedResult[]): Promise<ValidatedResult[]>`

생성 단계의 결과 배열을 받아 검증된 결과 배열을 반환하는 주 진입점.

**동작 단계**
1. `results`를 순차적으로(CPU 바운드 작업) 순회한다.
2. 결과에 `result.error`가 있거나 `result.components`가 없으면 `validationErrors: []`로 그대로 전달한다 (생성 실패 항목).
3. `result.components` 배열의 각 메시지에 대해 **스마트 AJV 검증**을 수행한다:
   - 메시지에 `createSurface` 키가 있으면 `CreateSurfaceMessage` 정의로, `updateComponents`이면 `UpdateComponentsMessage`로, `updateDataModel`이면 `UpdateDataModelMessage`로, `deleteSurface`이면 `DeleteSurfaceMessage`로 직접 검증한다.
   - 위 네 가지 키 중 아무것도 없으면 `validateFn`(최상위 oneOf)으로 폴백 검증한다.
4. AJV 검증 실패 시 **노이즈 제거 로직**을 수행한다:
   - 메시지 전체를 DFS로 탐색하여 `component` 문자열 필드를 가진 모든 오브젝트를 `pathToObject: Map<string, any>`에 수집한다 (경로 → 오브젝트). 순환 참조를 `visited: Set`으로 방지한다.
   - 각 컴포넌트 경로에 대해 `catalogs/basic/catalog.json#/components/{componentName}` 스키마로 개별 검증한다. 스키마를 찾을 수 없으면 "Unknown or hallucinated component type" 오류를 추가하고 해당 경로를 처리 완료로 표시한다.
   - 원래의 AJV 오류(`originalErrors`)에서 `oneOf`/`anyOf` 오류와, 이미 처리된 경로(또는 그 하위 경로)에 속하는 오류를 모두 제거한다.
   - 남은 구조적 오류와 컴포넌트별 오류를 합산하여 최종 `errors` 배열을 구성한다.
   - 각 오류 메시지를 포맷팅한다: `instancePath`를 역방향으로 탐색하여 가장 가까운 컴포넌트 타입 이름을 찾아 `(ComponentType)` 형태로 삽입하고, `params` 오브젝트가 있으면 들여쓰기 후 키=값으로 추가한다.
5. AJV 검증 완료 후 `validateCustom(components, errors)`를 호출하여 커스텀 검증을 수행한다.
6. 오류가 있으면 `failedCount`를 증가시키고, `outputDir`가 있으면 `saveFailure`를 호출한다.
7. `{...result, validationErrors: errors}`를 `validatedResults`에 추가한다.
8. 완료 후 통계(`Passed: N, Failed: M`)를 `logger.info`로 출력하고 결과를 반환한다.

---

#### `private saveFailure(result: GeneratedResult, errors: string[]): void`

검증 실패 결과를 YAML 파일로 저장한다.

- 저장 경로: `{outputDir}/output-{modelName}/details/{promptName}.{runNumber}.failed.yaml`
  - `modelName`의 `/`와 `:`는 `_`로 치환된다.
- 저장 내용: `{pass: false, reason: 'Schema validation failure', issues: [{issue, severity: 'criticalSchema'},...], overallSeverity: 'criticalSchema'}` 를 `yaml.dump`로 직렬화한다.

---

#### `private validateCustom(messages: any[], errors: string[]): void`

메시지 배열 전체에 대해 비즈니스 로직 검증을 수행한다.

**추적 변수**: `hasUpdateComponents`, `hasRootComponent`, `createdSurfaces: Set<string>`

**메시지별 처리**:
- `updateComponents`: `surfaceId`가 `createdSurfaces`에 없으면 오류 추가. `validateUpdateComponents` 호출. `components` 중 `id === 'root'`인 항목이 있으면 `hasRootComponent = true`.
- `createSurface`: `validateCreateSurface` 호출. `surfaceId`를 `createdSurfaces`에 추가.
- `updateDataModel`: `validateUpdateDataModel` 호출.
- `deleteSurface`: `validateDeleteSurface` 호출.
- 위 네 가지 키 모두 없음: "Unknown message type" 오류 추가.

**사후 검사**: `hasUpdateComponents && !hasRootComponent`이면 root 컴포넌트 누락 오류 추가.

마지막에 `validateFunctionCalls(messages, errors)`를 호출한다.

---

#### `private validateFunctionCalls(root: any, errors: string[]): void`

재귀적으로 오브젝트 트리를 탐색하여 FunctionCall 노드를 찾는다.

- 배열이면 각 항목에 재귀 호출한다.
- `root.call`이 문자열이고 키 개수가 2 또는 3개이면 FunctionCall로 판단한다.
  - `basicFunctions`에 포함되면 즉시 반환 (유효한 함수, 추가 오류 없음).
  - 미포함 함수는 현재 로직상 무시된다 (엄격한 스키마 검증에 위임).
- FunctionCall이 아니면 모든 프로퍼티 값에 재귀 호출한다.

---

#### `private validateCreateSurface(data: any, errors: string[]): void`

- `data.surfaceId`가 없으면 오류 추가.
- `data.catalogId`가 없으면 오류 추가.
- `['surfaceId', 'catalogId']` 외의 키가 있으면 각각 "unexpected property" 오류 추가.

#### `private validateDeleteSurface(data: any, errors: string[]): void`

- `data.surfaceId`가 없으면 오류 추가.
- `['surfaceId']` 외의 키가 있으면 "unexpected property" 오류 추가.

#### `private validateUpdateComponents(data: any, errors: string[]): void`

- `data.surfaceId`가 없으면 오류 추가.
- `data.components`가 없거나 배열이 아니면 오류 추가 후 조기 반환.
- 각 컴포넌트의 `id`에 대해 중복 검사를 수행하고, `componentIds: Set<string>`에 추가한다.
- `c.component`가 있으면 `catalogs/basic/catalog.json#/components/{componentType}` 스키마로 AJV 검증을 수행하고 오류를 수집한다.
- 모든 컴포넌트에 대해 `validateComponent`를 호출하여 참조 무결성을 확인한다.

#### `private validateUpdateDataModel(data: any, errors: string[]): void`

현재 구현에서는 스키마 검증에 완전히 위임하며 추가 검증 없이 빈 상태다. (v0.9에서 `op` 필드가 제거되어 단순화됨)

#### `private validateComponent(component: any, allIds: Set<string>, errors: string[]): void`

단일 컴포넌트의 참조 무결성을 검사한다.

- `id`가 없으면 오류 추가 후 반환.
- `component` (타입 문자열)가 없거나 string이 아니면 오류 추가 후 반환.
- `componentType`에 따라 `switch`로 분기하여 자식 참조를 확인한다:
  - `'Row'` / `'Column'` / `'List'`: `children`이 배열이면 `checkRefs(children)`, `children.componentId`가 있으면 `checkRefs([componentId])`.
  - `'Card'`: `checkRefs([component.child])`.
  - `'Tabs'`: 각 탭의 `tab.child`에 대해 `checkRefs`.
  - `'Modal'`: `checkRefs([component.trigger, component.content])`.
  - `'Button'`: `checkRefs([component.child])`.
- `checkRefs` 내부 헬퍼: 전달된 ID 배열 중 `allIds`에 없는 ID에 대해 "references non-existent component ID" 오류를 추가한다.

## 동작 흐름

평가 파이프라인의 두 번째 단계에서 `Validator` 인스턴스가 한 번 생성된다. 이후 `run(generatedResults)`를 호출하면, 각 결과의 `components` 배열을 순서대로 검증한다. 검증은 (1) AJV 스키마 검증 → (2) 노이즈 제거 및 컴포넌트별 재검증 → (3) 커스텀 비즈니스 로직 검증의 세 단계로 이루어지며, 모든 오류를 `validationErrors: string[]`로 취합한 `ValidatedResult` 배열을 반환한다.
