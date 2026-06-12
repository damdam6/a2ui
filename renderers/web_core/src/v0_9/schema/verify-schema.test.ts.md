# renderers/web_core/src/v0_9/schema/verify-schema.test.ts

## 개요

`server-to-client.ts`의 Zod 스키마를 실제 JSON Schema 사양 파일(`specification/v0_9/json/server_to_client.json`)과 구조적으로 비교하여 두 정의가 일치하는지 검증하는 테스트 파일이다. `zod-to-json-schema`로 Zod 스키마를 JSON Schema로 변환한 뒤, 정의(`$defs`) 수준과 최상위 `oneOf`/`properties` 수준에서 재귀적으로 비교한다. 일부 알려진 불일치(Zod 고유 아티팩트)는 명시적으로 무시한다.

## 의존성

### 외부 패키지
- `node:test` (`describe`, `it`)
- `node:assert`
- `zod-to-json-schema` (`zodToJsonSchema`)
- `fs` (`readFileSync`)
- `path` (`resolve`, `join`, `dirname`)
- `url` (`fileURLToPath`)

### 저장소 내부 모듈
- [`./server-to-client.js`](./server-to-client.ts.md) — `A2uiMessageSchema`, `CreateSurfaceMessageSchema`, `UpdateComponentsMessageSchema`, `UpdateDataModelMessageSchema`, `DeleteSurfaceMessageSchema`

## Exports

없음. 테스트 파일로 실행 전용이다.

## 테스트 케이스 명세

모든 테스트는 `describe('A2UI Schema Verification v0.9', ...)` 블록 내에 있다.

---

### `it('verifies v0.9 schema')`

**검증하는 동작**: Zod 스키마와 공식 JSON Schema 사양 파일이 구조적으로 일치하는지 확인한다.

**처리 흐름**:
1. `verifySchema('v0.9', A2uiMessageSchema, join(SPEC_DIR_V0_9, 'server_to_client.json'), { CreateSurfaceMessage, UpdateComponentsMessage, UpdateDataModelMessage, DeleteSurfaceMessage })`를 호출한다.
2. `zodToJsonSchema`를 통해 Zod 스키마를 JSON Schema 2019-09 포맷으로 변환하고, 4개 서브스키마를 `definitions`로 포함시킨다.
3. `readFileSync`로 공식 사양 파일을 읽어 파싱한다.
4. `compareDefinitions`로 `$defs` 수준의 차이를 계산하고, `assert.deepStrictEqual(diffs, {})`로 차이가 없음을 검증한다.
5. 공식 스키마에 `oneOf`/`anyOf`가 있으면 최상위 union 구조도 비교한다. `#/definitions/`를 `#/$defs/`로 정규화하여 비교한다.

**모킹/픽스처**: 파일시스템에서 `SPEC_DIR_V0_9` 경로의 `server_to_client.json`을 읽는다. `__dirname`은 컴파일된 `dist/src/v0_9/schema` 디렉토리로 해석되며, 그로부터 6단계 상위 디렉토리의 `specification/v0_9/json`을 `resolve`로 계산한다.

---

### `it('validates A2uiMessage wrapper')`

**검증하는 동작**: `A2uiMessageSchema.parse()`가 `deleteSurface` 메시지를 올바르게 파싱하고 원본과 동일한 객체를 반환하는지 확인한다.

**픽스처**: `{ version: 'v0.9', deleteSurface: { surfaceId: 'surface-1' } }`.

**검증 방법**: `assert.deepStrictEqual(A2uiMessageSchema.parse(msg), msg)`로 파싱 결과가 입력과 동일한지 확인한다.

## 헬퍼 함수 명세

### `compareDefinitions(zodDefs, jsonDefs): Record<string, any>`

두 `$defs` 객체를 비교하여 차이 맵을 반환한다.

- 두 객체의 키 집합을 합산(`Set`)하여 순회한다.
- `'A2uiMessage'` 키는 Zod 래퍼 아티팩트이므로 건너뛴다.
- 한쪽에만 있으면 `{ error: 'Missing in ...' }`를 기록한다.
- 양쪽 모두 있으면 `getObjectDiff`로 구조적 차이를 계산하여 빈 객체가 아닌 경우 기록한다.

### `getObjectDiff(obj1, obj2, path = ''): Record<string, any>`

두 객체를 재귀적으로 비교하여 경로-차이 맵을 반환한다.

- `'description'`, `'title'`, `'$id'`, `'$schema'` 키는 무시한다.
- `path`가 `'version'`으로 끝나고 `key === 'type'`, `val1 === 'string'`, `val2 === undefined`이면 무시한다 (Zod가 const에 `type: "string"`을 방출하지만 JSON Schema에는 없는 경우).
- `currentPath`가 `'updateDataModel.properties.value.additionalProperties'`로 끝나고 `val1 === undefined`, `val2 === true`이면 무시한다 (Zod 기본값 차이).
- `currentPath`에 `'components.items'`가 포함되면 무시한다 (Zod는 `AnyComponentSchema`를 인라인 해석하지만 JSON 사양은 `$ref`를 사용).
- `currentPath`에 `'theme.$ref'`가 포함되고 `val1 === undefined`, `val2 === 'catalog.json#/$defs/theme'`이면 무시한다 (Zod는 `any`로 정의).
- 두 값 모두 배열이면 정렬 후 `JSON.stringify` 비교를 수행한다.
- 두 값 모두 객체이면 재귀 호출한다.
- 그 외 값이 다르면 `{ zod: val1, json: val2 }`를 기록한다.

### `verifySchema(version, zodSchemaSpec, jsonSpecPath, definitionsMap?)`

실제 비교 로직을 수행하는 헬퍼 함수. `describe` 블록 내 테스트들이 호출한다.

1. `zodToJsonSchema`로 Zod → JSON Schema 변환 (포맷: `jsonSchema2019-09`, 루트 이름: `'A2uiMessage'`).
2. `readFileSync`로 공식 사양 로드.
3. `definitionsMap`이 있으면 `compareDefinitions`로 `$defs` 비교 후 `assert.deepStrictEqual(diffs, {})`로 검증.
4. 공식 스키마의 최상위 구조에 따라:
   - `oneOf`/`anyOf`가 있으면 Zod의 `anyOf`/`oneOf`와 비교 (`#/definitions/`를 `#/$defs/`로 정규화).
   - `properties`가 있으면 Zod의 `properties`와 비교.
5. 모든 비교 통과 시 콘솔에 성공 메시지를 출력한다.

## 동작 흐름

1. 모듈 수준에서 `__dirname`을 ESM 방식(`fileURLToPath(import.meta.url)`)으로 계산하고, `SPEC_DIR_V0_9`를 상대 경로로 해석한다.
2. 테스트 실행 시 v0.9 스키마를 파일시스템의 사양 파일과 비교 검증하고, 단순 파싱 성공도 함께 확인한다.
