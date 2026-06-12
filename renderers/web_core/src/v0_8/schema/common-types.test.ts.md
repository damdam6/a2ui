# renderers/web_core/src/v0_8/schema/common-types.test.ts

## 개요

`DataValueSchema`의 재귀 깊이 제한 동작을 검증하는 단위 테스트 파일이다. `node:test`와 `node:assert`를 사용하며, `DataValueSchema`(기본 깊이 5)와 `createDataValueSchema`(커스텀 깊이)가 깊이 조건에 따라 올바르게 통과/거부하는지 확인한다.

## 의존성

### 외부 패키지
- `node:test` — `describe`, `it`
- `node:assert` — `assert.strictEqual`

### 저장소 내부 모듈
- [`./common-types.js`](./common-types.ts.md) — `DataValueSchema`, `createDataValueSchema`

## Exports

(없음 — 테스트 파일은 어떤 것도 export하지 않는다)

## 테스트 케이스 명세

### `describe('DataValueSchema recursion depth')`

#### 테스트 1: `'should allow depth <= 5 by default'`

- **검증 동작**: `DataValueSchema`의 기본 최대 깊이가 5임을 확인한다. `key`/`valueMap` 중첩이 5단계인 데이터(`root → level2 → level3 → level4 → level5 (valueString: 'leaf')`)를 `safeParse`하면 `result.success === true`여야 한다.
- **픽스처**: 5단계 `valueMap` 중첩 인라인 객체 리터럴. 마지막 노드는 `valueString: 'leaf'`.
- **모킹**: 없음.

#### 테스트 2: `'should reject depth > 5 by default'`

- **검증 동작**: 6단계 중첩(`root → level2 → level3 → level4 → level5 → level6`)은 기본 깊이 제한을 초과하므로 `result.success === false`여야 한다. 또한 `result.error.issues[0].message`가 정확히 `'valueMap recursion exceeded maximum depth of 5.'`이어야 한다.
- **픽스처**: 6단계 `valueMap` 중첩 인라인 객체 리터럴.
- **모킹**: 없음.

#### 테스트 3: `'should allow overriding depth limit'`

- **검증 동작**: `createDataValueSchema({ maxDepth: 2 })`로 생성한 커스텀 스키마가 깊이 제한 2를 올바르게 적용하는지 확인한다.
  - 2단계 중첩(`root → level2 (valueString: 'leaf')`)은 통과해야 한다(`validResult.success === true`).
  - 3단계 중첩(`root → level2 → level3 (valueString: 'leaf')`)은 실패해야 한다(`invalidResult.success === false`).
  - 실패 시 `invalidResult.error.issues[0].message`가 `'valueMap recursion exceeded maximum depth of 2.'`이어야 한다.
- **픽스처**: `CustomSchema`(`createDataValueSchema({ maxDepth: 2 })`), 2단계 유효 데이터, 3단계 유효하지 않은 데이터 — 모두 인라인 객체 리터럴.
- **모킹**: 없음.

## 동작 흐름

테스트는 독립적이며 각각 인라인 픽스처를 사용한다. 외부 상태나 모킹 없이 `safeParse`의 반환값 구조(`success`, `error.issues[0].message`)만을 단언한다.
