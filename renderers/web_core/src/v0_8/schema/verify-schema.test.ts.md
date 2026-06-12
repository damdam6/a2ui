# renderers/web_core/src/v0_8/schema/verify-schema.test.ts

## 개요

A2UI v0.8의 Zod 스키마(`A2uiMessageSchema`)가 공식 JSON Schema 사양 파일과 구조적으로 일치하는지 검증하는 테스트 파일이다. `zod-to-json-schema`로 Zod 스키마를 JSON Schema로 변환한 뒤 디스크의 공식 명세 파일과 재귀적으로 비교한다. Zod와 JSON Schema 간의 알려진 표현 차이(예: 재귀 참조 방식, `AnyComponentSchema` 인라인 해석 등)는 명시적으로 무시하도록 처리된다.

## 의존성

### 외부 패키지
- `node:test` — `describe`, `it`
- `node:assert` — `deepStrictEqual`
- `zod-to-json-schema` — `zodToJsonSchema`
- `fs` — `readFileSync`
- `path` — `resolve`, `join`, `dirname`
- `url` — `fileURLToPath`

### 저장소 내부 모듈
- [`./server-to-client.ts`](./server-to-client.ts.md) — `A2uiMessageSchema` (`V08A2uiMessageSchema`로 import)

## Exports

없음 (테스트 파일)

## 테스트 케이스

### describe: `A2UI Schema Verification v0.8`

#### it: `'verifies v0.8 schema'`

**픽스처/설정**:
- `SPEC_DIR_V0_8`: 현재 파일의 `__dirname`에서 6단계 상위로 올라가 `specification/v0_8/json` 디렉토리를 찾는다. (`__dirname`은 빌드 결과인 `dist/src/v0_8/schema`를 가리킨다.)
- 비교 대상 파일: `server_to_client_with_standard_catalog.json`

**검증 동작**:
`verifySchema('v0.8', V08A2uiMessageSchema, <jsonSpecPath>)`를 인수 없이 호출한다. `definitionsMap`이 제공되지 않으므로 정의 레벨 비교(`compareDefinitions`)는 건너뛰고, 최상위 `properties` 비교만 수행된다.

---

### 비공개 함수: `verifySchema`

시그니처: `(version: string, zodSchemaSpec: any, jsonSpecPath: string, definitionsMap?: Record<string, any>) => void`

1. `zodToJsonSchema(zodSchemaSpec, { target: 'jsonSchema2019-09', definitions: definitionsMap || {}, name: 'A2uiMessage' })`로 Zod 스키마를 JSON Schema 문자열로 직렬화한다.
2. `readFileSync(jsonSpecPath, 'utf-8')`로 공식 명세 파일을 읽는다.
3. 두 문자열을 `JSON.parse`로 파싱한다.
4. `definitionsMap`이 제공된 경우, 생성된 스키마와 공식 스키마의 `$defs`/`definitions`를 꺼내 `compareDefinitions`로 비교한다. 차이가 있으면 `assert.deepStrictEqual(diffs, {}, ...)`로 실패를 유발한다.
5. `generatedSchema`에서 `definitions['A2uiMessage']` 또는 `$defs['A2uiMessage']`를 루트 Zod 스키마로 추출한다.
6. 공식 스키마가 `oneOf`/`anyOf` 구조이면: Zod 스키마의 `anyOf`/`oneOf`를 가져와 `#/definitions/` 접두어를 `#/$defs/`로 정규화한 뒤 `getObjectDiff`로 비교한다.
7. 공식 스키마가 `properties` 구조이면: Zod 스키마의 `properties`와 직접 `getObjectDiff`로 비교한다.
8. 차이가 있으면 `assert.deepStrictEqual`로 실패를 유발한다.
9. 성공 시 `console.log`로 확인 메시지를 출력한다.

---

### 비공개 함수: `compareDefinitions`

시그니처: `(zodDefs: any, jsonDefs: any) => Record<string, any>`

두 정의 맵의 키 합집합을 순회한다. `'A2uiMessage'` 키는 Zod 래퍼 아티팩트이므로 건너뛴다. 한쪽에만 키가 있으면 `{ error: 'Missing in Zod schema' }` 또는 `{ error: 'Missing in JSON schema' }` 를 기록한다. 양쪽에 모두 존재하면 `getObjectDiff`로 세부 비교하여 차이가 있을 때만 결과 맵에 포함한다. 최종적으로 발견된 차이 맵을 반환한다.

---

### 비공개 함수: `getObjectDiff`

시그니처: `(obj1: any, obj2: any, path?: string) => Record<string, any>`

두 객체를 재귀적으로 비교하여 구조적 차이를 dotted path 형태의 키로 반환한다. `description`, `title`, `$id`, `$schema` 키는 무시(`ignoreKeys`)한다. 배열인 경우 정렬 후 `JSON.stringify`로 비교한다. 중첩 객체는 재귀 호출로 처리한다.

**알려진 차이를 명시적으로 건너뛰는 규칙들**:
- `path`가 `version`으로 끝나고 `key === 'type'`이며 `val1 === 'string'`이고 `val2 === undefined`인 경우: Zod는 const에 `type: "string"`을 내보내지만 JSON Schema는 생략하므로 무시.
- `currentPath`가 `updateDataModel.properties.value.additionalProperties`로 끝나고 `val1 === undefined`, `val2 === true`인 경우: Zod 기본값 차이이므로 무시.
- `currentPath`에 `components.items`가 포함된 경우: Zod는 `AnyComponentSchema`를 인라인 해석하지만 JSON 명세는 `catalog.json`의 `$ref`를 사용하므로 무시.
- `currentPath`에 `theme.$ref`가 포함되고 `val1 === undefined`, `val2 === 'catalog.json#/$defs/theme'`인 경우: 무시.
- `currentPath`에 `valueMap.items.properties.valueMap`이 포함된 경우: v0.8 JSON 명세가 재귀를 의도적으로 생략했으므로 무시.
- `currentPath`에 `dataModelUpdate.properties.contents.items.properties.valueMap.items`가 포함된 경우: Zod의 재귀 `$ref` 표현 방식 차이이므로 무시.

## 동작 흐름

테스트 실행 시 `node --test dist/**/*.test.js` 명령으로 컴파일된 파일이 호출된다. `__filename`과 `__dirname`이 런타임에 `fileURLToPath`와 `dirname`으로 복원된 뒤, `SPEC_DIR_V0_8` 경로가 계산된다. 테스트 스위트가 등록되고, `verifySchema`가 호출되어 Zod → JSON Schema 변환, 파일 읽기, 재귀 비교가 순서대로 수행되며 차이가 없으면 테스트가 통과된다.
